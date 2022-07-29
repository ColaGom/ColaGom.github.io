---
layout: post
title: "Spring 애플 로그인(oauth) 구현"
categories: [Dev, Spring]
tags: [spring, jpa, oauth, apple]

---

# Sign in with apple server side

![flow-apple-auth](/assets/img/220729-1-1.png)

1. Client에서 구현된 oauth webview or login sdk 를 활용하여 signin with apple
2. `authorization_code` 획득
3. sign request with `authorization_code`

---

### Server side

1. `authorization_code` 를 통해 token 생성 요청
2. `TokenResponse`
3. `TokenResponse` validate
4. `TokenResponse` 에서 UserInfo 추출

> 4~7 과정의 내용을 정리
>

# 사전준비

- JWT 관련 기능은 auth0 library를 사용
- **Certificates, Identifiers & Profiles**
  - team-id, client-id
  - key-id
  - private key file
  - 모두 apple developer console에서 획득 가능하다

# 주의사항

> ***`authorization_code` is single-use only***
>

> *server-side authenticate이므로 Token verification 구현은 생략해도되나 참고용으로 작성*
>

# 결과

```kotlin
class ApplePublicKeys(
    val keys: Array<ApplePublicKey>
)

data class ApplePublicKey(
    val alg: String,
    val e: String,
    val kid: String,
    val kty: String,
    val n: String,
    val use: String
)

data class TokenResponse(
    val accessToken: String,
    val expiresIn: Int,
    val idToken: String,
    val refreshToken: String,
    val tokenType: String
)

/**
 * Token generate에 사용할 client-secret은 ECDSA (비대칭 암호화) algorithm을 사용하여
 * 생성해야된다.
 */
class ExampleKeyProvider : ECDSAKeyProvider {

    private val PEM_URI = ""

    /**
     * singing 과정만 필요하므로 private key만 구현
     */
    override fun getPublicKeyById(keyId: String?): ECPublicKey? = null

    override fun getPrivateKey(): ECPrivateKey {
        val file = ResourceUtils.getFile(PEM_URI)
        PemReader(FileReader(file)).use { reader ->
            val content = reader.readPemObject().content
            return KeyFactory.getInstance("EC")
                .generatePrivate(PKCS8EncodedKeySpec(content)) as ECPrivateKey
        }
    }

    override fun getPrivateKeyId(): String? = null
}

object AppleAuthExample {
    private val rest by lazy {
        RestTemplate()
    }
    private val CLIENT_ID = ""
    private val TEAM_ID = ""
    private val KEY_ID = ""
    private const val AUTH_URL = "https://appleid.apple.com"

    fun createClientSecret(): String? {
        val now = Date()
        return JWT.create()
            .withHeader(
                mapOf(
                    "kid" to KEY_ID
                )
            )
            .withSubject(CLIENT_ID)
            .withIssuer(TEAM_ID)
            .withIssuedAt(now)
            .withExpiresAt(Date(now.time + 1.hours.inWholeMilliseconds))
            .withAudience(AUTH_URL)
            .sign(Algorithm.ECDSA256(ExampleKeyProvider()))
    }

    fun authenticate(authCode: String) {
        val headers = HttpHeaders().apply {
            contentType = MediaType.APPLICATION_FORM_URLENCODED
        }

        val map: MultiValueMap<String, String> = LinkedMultiValueMap<String, String>().apply {
            add("client_id", CLIENT_ID)
            add("client_secret", createClientSecret())
            add("grant_type", "authorization_code")
            add("code", authCode)
        }

        val entity = HttpEntity(map, headers)

        //authorization_code를 통해 token을 생성한다
        val idToken = rest.exchange(
            "https://appleid.apple.com/auth/token",
            HttpMethod.POST,
            entity,
            TokenResponse::class.java
        ).body?.idToken ?: throw RuntimeException("invalid or revoked authorization_code")

        val jwt = decodeIdToken(idToken) ?: throw RuntimeException("invalid idToken")
        if (!verifyToken(jwt)) throw RuntimeException("invalid idToken")

        /**
         * TODO: handle singing
         * jwt.subject is unique userId
         * jwt.getClaim("email") is user email
         */
    }

    //Verify the JWS E256 signature using the server’s public key
    private fun decodeIdToken(token: String?): DecodedJWT? {
        val keys =
            rest.getForEntity("https://appleid.apple.com/auth/keys", ApplePublicKeys::class.java).body?.keys
                ?: return null

        keys.forEach {
            val nBytes: ByteArray = Base64.getUrlDecoder().decode(it.n)
            val eBytes: ByteArray = Base64.getUrlDecoder().decode(it.e)
            val modules = BigInteger(1, nBytes)
            val exponent = BigInteger(1, eBytes)

            val spec = RSAPublicKeySpec(modules, exponent)
            val kf: KeyFactory = KeyFactory.getInstance("RSA")
            val publicKey: RSAPublicKey = kf.generatePublic(spec) as RSAPublicKey

            try {
                return JWT.require(
                    Algorithm.RSA256(
                        publicKey, null
                    )
                ).build().verify(token)
            } catch (e: Exception) {
            }
        }
        return null
    }

    /**
     * https://developer.apple.com/documentation/sign_in_with_apple/sign_in_with_apple_rest_api/verifying_a_user
     */
    private fun verifyToken(token: DecodedJWT): Boolean {
        //Verify that the time is earlier than the exp value of the token
        val verifyTime = Date() < token.expiresAt

        //Verify that the aud field is the developer’s client_id
        val verifyAud = token.audience.firstOrNull() == true

        //Verify that the iss field contains https://appleid.apple.com
        val verifyIssuer = token.issuer == AUTH_URL

        return verifyTime && verifyAud && verifyIssuer
    }
}
```
# 참조

[Apple Developer Documentation](https://developer.apple.com/documentation/sign_in_with_apple)

[https://github.com/auth0/java-jwt](https://github.com/auth0/java-jwt)
