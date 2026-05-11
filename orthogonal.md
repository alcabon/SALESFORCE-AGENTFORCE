The answer to this requires unpacking a critical conceptual split first — `grant_type` and `client_assertion_type` are **orthogonal axes**, not variants of the same thing. One controls *what you're exchanging*, the other controls *how the client proves its own identity*. They can be combined freely.The mental model that unlocks all of this:

<img width="1472" height="2872" alt="image" src="https://github.com/user-attachments/assets/4da3bea1-cf92-4a0a-8dd2-d44a5f369da9" />


**`grant_type`** answers "what credential are you trading in?" — it's the exchange mechanism. **`client_assertion_type`** answers "but first, prove to me you're the client you claim to be" — it's the door pass before any exchange happens. Because they're orthogonal, `client_assertion_type=jwt-bearer` can appear alongside any `grant_type`.

The sharpest distinction, which trips up most people:

`grant_type=urn:ietf:params:oauth:grant-type:jwt-bearer` puts the JWT in the `assertion` parameter and the JWT **represents a subject** (a user, a service account). The auth server is being asked "issue a token that acts as this person/system." This is what Salesforce uses in the classic **JWT Bearer Flow** for user context (used before External Credentials existed).

`client_assertion_type=urn:ietf:params:oauth:client-assertion-type:jwt-bearer` puts the JWT in the `client_assertion` parameter and the JWT **represents the client application**. The auth server is being asked "I trust that you are the registered app." This is what Salesforce's **Client Credentials with JWT Assertion** external credential uses.

The "two JWTs" row is the most powerful combination — `grant_type=jwt-bearer` + `client_assertion_type=jwt-bearer` — where you send two separate signed JWTs in the same request: one asserting *who you're acting as*, one proving *who you are*. FAPI 2.0 (Financial-grade API) mandates this pattern for high-assurance financial services integrations.



---

Exactly right. The asymmetry is intentional — when Salesforce is the **authorization server** (call-in), it owns both sides of the contract and can hide the complexity. When it's the **client** (call-out), it has to speak whatever dialect the external IdP demands, hence all those External Credential fields.The core reason for the simplicity gap: on the call-in side, Salesforce controls the entire authorization server stack — the token endpoint URL is always `https://login.salesforce.com/services/oauth2/token`, the JWT validation algorithm is fixed (RS256), the claim requirements are published in the docs. The external caller simply follows Salesforce's spec.

<img width="1472" height="2216" alt="image" src="https://github.com/user-attachments/assets/1fbb9d39-3950-40cc-a68a-2b70b301694f" />

On the call-out side, every external IdP is different — Azure AD needs a `resource` param, Okta has its own `aud` format, some servers expect the `client_id` in the POST body and some in the header. Salesforce's External Credential fields exist to absorb all that variation.

The cleanest call-in flow from a configuration standpoint is **JWT Bearer**: you upload one certificate, enable the scopes, and that's it. The external system builds a JWT with four mandatory claims (`iss`, `sub`, `aud`, `exp`) and Salesforce handles the rest — no secret to rotate, no token URL to remember, no additional parameters. The **Client Credentials** flow (Spring '23+) is even simpler to set up — just a key + secret and a designated "Run As" user — but it trades the certificate-based security for a shared secret, which needs rotation management.
