# CISC 820 — Foundations of Privacy: Study Notes

Based on Topics 1–9 and the supplied sample quiz. Explanations and worked examples are added for studying. These notes distinguish the goal of a technique from the assumptions needed for its guarantee; encryption alone does not make every surrounding protocol private.

## How the topics fit together

| Question | Relevant technique |
|---|---|
| Can released statistics hide one person's influence? | Differential privacy (DP) |
| Can users protect their values before submitting them? | Local differential privacy (LDP) |
| Can a published table resist linking attacks? | k-anonymity, l-diversity, t-closeness |
| Can someone compute on encrypted values? | Homomorphic encryption |
| Can communication hide who talks to whom? | Mix networks, onion routing, DC-nets |
| Can a server answer without learning which item I want? | Private information retrieval (PIR) |
| Can I learn one selected item while other items stay secret? | Oblivious transfer (OT) |
| Can I prove knowledge without revealing the secret? | Zero-knowledge proof (ZKP) |
| Can two users discover a match without exposing all their data? | Private equality, inclusion, intersection |

## Notation reference

| Symbol | Meaning |
|---|---|
| D, D′ | Databases; D ∼ D′ means adjacent |
| ε | Privacy parameter; smaller generally means stronger privacy |
| S(F), ΔF | Global sensitivity of query F |
| E(m;r), Dec(c) | Encryption of m using randomness r; decryption |
| pk, sk | Public and secret keys |
| a ≡ b mod n | n divides a−b |
| a⁻¹ mod n | Multiplicative inverse, existing iff gcd(a,n)=1 |
| ⊕ | Bitwise XOR, or addition modulo 2 |
| c·q | Dot product: multiply corresponding entries and add |
| Zₙ, Zₙ* | Integers modulo n; invertible integers modulo n |

Letters are reused across topics. In Bloom-filter formulas below, M is the number of bits and t the number of inserted items. In RSA and Paillier, n is a public modulus. In frequency estimation, n is the number of users.

## 1. Database privacy

### Security and privacy

Security protects a system and its data from unauthorized access or alteration. Privacy also concerns what people can infer about individuals from authorized releases.

| Property | Meaning | Example |
|---|---|---|
| Availability | Data remain accessible when required | Authorized staff can access needed records |
| Authenticity | Identities and origins are genuine | Verify who submitted a record |
| Integrity | Data are not improperly altered | Detect changed medical records |
| Confidentiality | Unauthorized parties cannot read protected data | Restrict salary access |

Authentication factors: **what you know** (password), **what you have** (token), **what you are** (biometric). Biometrics can have false acceptance and false rejection. Access control, encryption, auditing, masking, and monitoring address different threats; none alone prevents inference from released statistics.

### Identifiers, quasi-identifiers, and sensitive attributes

- **Identifier:** directly identifies someone, such as a student number.
- **Quasi-identifier (QI):** a combination such as birth date, sex, and postal code can identify someone when linked to outside data.
- **Sensitive attribute (SA):** a value to protect, such as diagnosis or salary.

Removing names is not enough. A public table may map a name to QIs, and a released private table may map the same QIs to a diagnosis. Joining them reveals the diagnosis. The AOL example illustrates how even pseudonymous search histories can expose identities through their content.

**Two disclosure goals:** prevent linking an individual to a sensitive attribute, and prevent learning that an individual is present in a dataset. Membership itself can be sensitive.

### Release models and utility

Interactive release uses a gatekeeper that answers or rejects queries. Offline release publishes a transformed dataset. A “send me your code” model runs external analysis at the data owner, but requires controlling malicious code and output leakage.

Privacy competes with utility. Empty data are private but useless. Utility can be measured by information loss, rows suppressed, query error, or performance on a representative workload.

### Anonymization methods

| Method | Operation | Main tradeoff |
|---|---|---|
| Suppression | Remove entries, attributes, or rows | Missing information |
| Generalization | Replace exact values by ranges or categories | Less precise queries |
| Permutation/swapping | Reassign SA values within groups | Preserves marginal values but changes associations |
| Perturbation | Add noise or otherwise modify values | Distorted individual values |
| Synthesis | Generate artificial records | Quality and privacy depend on the generator |

Synthetic data are not automatically differentially private.

### k-anonymity

A table is k-anonymous with respect to selected QIs if **every QI pattern occurs at least k times**. Records sharing a QI pattern form an equivalence class.

Example: three rows all show age 20–29 and postal code K7*. They form a class of size three. A table is 3-anonymous only if every class has size at least three.

To check k-anonymity: select the QIs, group equal released QI values, count each group, and take the minimum group size. Do not group on the SA when performing this test.

**Limitation:** if all three people have the same diagnosis, knowing the target belongs to this class reveals that diagnosis. This is a homogeneity attack. Background knowledge can also narrow the possibilities.

### l-diversity and t-closeness

| Model | Requirement within each QI class | What it addresses |
|---|---|---|
| k-anonymity | At least k records | Unique linking through the selected QIs |
| l-diversity | At least l well-represented SA values | Homogeneous sensitive values |
| t-closeness | SA distribution is within distance t of the full-table SA distribution | Distributional and semantic disclosure |

Simply counting distinct values is the simplest diversity variant; “well-represented” can impose stronger conditions. Three distinct salaries of $50,000, $50,001, and $50,002 provide little uncertainty about salary despite being distinct. Smaller t is a stricter closeness requirement.

### Incognito and possible worlds

Incognito searches a lattice of full-domain generalizations for minimal k-anonymous releases. Two useful pruning rules:

1. **Subset property:** k-anonymity for Q implies k-anonymity for any subset of Q.
2. **Generalization property:** further generalizing a k-anonymous table preserves k-anonymity.

Minimal means no strictly less-generalized candidate below it satisfies the requirement; several incomparable minimal solutions can exist. A utility metric chooses among them.

A generalized table represents many possible original tables, or **possible worlds**. A fact is possible if true in some world and certain if true in every world. Treating worlds as equally likely requires a modeling assumption; it is not automatic.

Permutation preserves the multiset of SA values in a group. Therefore, assignments to different records are dependent: assigning one salary uses up one occurrence. Minimality and deFinetti-style attacks exploit knowledge of the algorithm or statistical relationships to narrow plausible worlds.

### Differential privacy

DP limits how much a release distribution changes because of one person's participation or record.

For every adjacent D,D′ and every output event T:

\[
\Pr[A(D)\in T]\le e^\epsilon\Pr[A(D')\in T].
\]

Because adjacency is symmetric, the reversed inequality also applies. Small ε makes the distributions harder to distinguish. DP limits additional disclosure attributable to a person's data; it does not hide every population-level fact or guarantee that no inference can ever be made.

**Adjacency must be specified:** add/remove-one adjacency changes whether a person is present; replacement adjacency replaces one record. Sensitivity can differ between these models.

### Global sensitivity

\[
S(F)=\max_{D\sim D'}\|F(D)-F(D')\|_1,
\qquad \|v\|_1=\sum_i|v_i|.
\]

Sensitivity is the worst possible change, not the average observed change.

| Query | Sensitivity / condition |
|---|---|
| Count of records satisfying a predicate | 1 |
| Histogram under add/remove adjacency | 1 in L₁ distance |
| Histogram under replacement adjacency | At most 2: one bin loses 1, another gains 1 |
| Sum with fixed-size replacement and x∈[a,b] | b−a |
| Mean with fixed size n, replacement, x∈[a,b] | (b−a)/n |
| Minimum or maximum on a bounded range [a,b] | Can be as large as b−a |
| Minimum, maximum, or sum without bounds | May be unbounded |

The slides use sensitivity 2 for a histogram in the replacement example and 1 for a disjoint-cell add/remove example. These are different adjacency assumptions, not competing arithmetic answers.

### Laplace mechanism and proof

Release A(D)=F(D)+Z, where

\[
Z\sim\operatorname{Lap}(b),\qquad b=S(F)/\epsilon,
\qquad f_Z(z)=\frac{1}{2b}e^{-|z|/b}.
\]

Facts: E[Z]=0, E[|Z|]=b, Var(Z)=2b². Zero mean does not mean the noise has zero magnitude.

Example: a count with S=1 and ε=0.5 uses b=2, so expected absolute noise is 2. Increasing sensitivity or decreasing ε increases noise.

For a vector query, add independent Laplace noise to coordinates calibrated to the vector's L₁ sensitivity. At output t, the density ratio is

\[
\frac{f_{A(D)}(t)}{f_{A(D')}(t)}
=\exp\left(\frac{\epsilon}{S(F)}
[\|F(D')-t\|_1-\|F(D)-t\|_1]\right)
\le e^\epsilon.
\]

The reverse triangle inequality bounds the bracket by ‖F(D′)−F(D)‖₁, which is at most S(F). For continuous distributions, this calculation uses densities; probabilities of individual exact real outputs are generally zero. Integrating gives the event-based DP guarantee.

### Symmetric geometric mechanism

For integer outputs, release F(D)+Z with

\[
\alpha=e^{\epsilon/S(F)}>1,
\qquad \Pr[Z=k]=\frac{\alpha-1}{\alpha+1}\alpha^{-|k|},\quad k\in\mathbb Z.
\]

Larger S or smaller ε makes α closer to 1, giving slower-decaying tails and more noise. Its privacy proof uses the same reverse-triangle argument, giving a probability ratio at most α^S=e^ε. Do not confuse this α>1 convention with formulations using its reciprocal.

### Composition and post-processing

- **Sequential composition:** repeated mechanisms on the same person's data cost at most ε₁+⋯+εₖ under basic pure-DP composition.
- **Parallel composition:** on disjoint groups of people under the appropriate adjacency/partition assumptions, the cost is maxᵢ εᵢ.
- **Post-processing:** computing on a private output without new private-data access costs no additional privacy budget.

Example: three queries about the same dataset, each ε=0.2, give a basic total of 0.6. The same queries on three disjoint groups give 0.2 under add/remove adjacency. Moving a person between groups under replacement adjacency needs care.

The basic sequential bound also supports appropriately private adaptive mechanisms; independence is not a universal requirement. Sublinear advanced bounds require additional conditions and usually approximate-DP accounting, rather than a blanket improvement for pure DP.

## 2. Big data privacy and local DP

### Background and protections

The three Vs are **volume** (scale), **velocity** (generation/processing speed), and **variety** (heterogeneous formats). The four As describe **acquisition, aggregation, analysis, application**. Threats arise throughout this lifecycle, not only in stored databases.

The lecture distinguishes limited disclosure, data transformation, procedural safeguards, and cryptographic controls. Examples include aggregation, suppression, audit logs, MPC, and homomorphic encryption. Secure outsourced computation also needs a way to verify correctness cheaply. Federated learning alone is not a formal privacy guarantee; model updates can leak information.

### Central DP versus local DP

| Feature | Central DP | LDP |
|---|---|---|
| Protected input | One person's contribution to a database | Any two possible values of one user's input |
| Randomization | Trusted curator releases a noisy result | User randomizes before collection |
| Server sees raw values? | Curator may see them | Only randomized reports |
| Typical count error, fixed privacy | Independent of n | Grows on the order of √n in standard settings |
| Trust/utility tradeoff | Requires trust in curator | Usually less accurate |

LDP definition:

\[
\Pr[A(x)=y]\le e^\epsilon\Pr[A(x')=y]
\quad\text{for every }x,x',y.
\]

For small ε, representative count-error scales are O(1/ε) centrally and O(√n/ε) locally. A frequency/proportion divides the count by n, so its error also divides by n. Counts and proportions must not be compared as if they were the same unit.

Post-processing is free in LDP. Repeated reports by the same user compose. Splitting one person's record into fields does not automatically grant parallel composition; using disjoint groups of users is different.

### Frequency oracle: Encode → Perturb → Estimate

Encode maps value v to a representation. Perturb randomizes it locally. Estimate combines reports at the server. The server should not collect raw values and then call that local privacy.

### Binary randomized response (BRR)

For x∈{0,1}, report x with probability p and the opposite with probability q:

\[
p=\frac{e^\epsilon}{e^\epsilon+1},\qquad
q=\frac{1}{e^\epsilon+1},\qquad p/q=e^\epsilon.
\]

The coin example says “tell the truth with probability 1/2; otherwise answer with a fair random bit.” Overall truth probability is 1/2+(1/2)(1/2)=3/4, not 1/2. Thus p=0.75, q=0.25, and ε=ln 3.

If nᵥ users have true value v and Iᵥ reports equal v, then

\[
E[I_v]=n_vp+(n-n_v)q=n_v(p-q)+nq,
\qquad \widehat n_v=\frac{I_v-nq}{p-q}.
\]

Example: n=100, p=0.75, q=0.25, and Iᵥ=40 gives estimated count (40−25)/0.5=30. The estimate removes expected false positives and corrects the signal attenuation.

An unbiased estimate can fall outside [0,n]. Clipping is free post-processing but can introduce bias.

### GRR, SUE, and OUE

| Mechanism | Encoding/report | p | q |
|---|---|---|---|
| GRR | One categorical value from domain size d | e^ε/(e^ε+d−1) | 1/(e^ε+d−1) for each incorrect value |
| SUE | Perturbed one-hot d-bit vector | e^(ε/2)/(e^(ε/2)+1) | 1/(e^(ε/2)+1) |
| OUE | Perturbed one-hot d-bit vector | 1/2 | 1/(e^ε+1) |

For UE, p=P(output bit 1 | input bit 1) and q=P(output bit 1 | input bit 0). The perturbed vector need not remain one-hot. All use the debiasing estimator (Iᵥ−nq)/(p−q).

**Why ε/2 in SUE?** Different one-hot inputs differ at two positions. The maximum likelihood ratio is

\[
\frac{p(1-q)}{q(1-p)}\le e^\epsilon.
\]

For SUE, the two changed positions each contribute e^(ε/2). OUE satisfies the same total bound with asymmetric probabilities.

GRR communicates roughly log₂d bits but its variance deteriorates with a large domain. Unary encoding communicates d bits in this basic form; per-coordinate variance can avoid linear dependence on d. This does not mean the total error across all coordinates is independent of d.

### Variance: keep the sampling model straight

For a fixed dataset containing nᵥ occurrences, reports are independent but generally have different Bernoulli parameters p or q. The exact conditional variance is

\[
\operatorname{Var}(\widehat n_v\mid\text{fixed data})
=\frac{n_vp(1-p)+(n-n_v)q(1-q)}{(p-q)^2}.
\]

If user values themselves are independent draws from a population with frequency f, each report has marginal probability P*=fp+(1−f)q. Then the reported count is Binomial(n,P*) and its variance is nP*(1−P*). This includes sampling variability. Independence alone does not make unequal-probability reports identically distributed.

For rare values, nᵥ≪n, useful approximations are:

| Mechanism | Approximate conditional count variance |
|---|---|
| BRR | n e^ε/(e^ε−1)²; in binary RR this is also exact for any fixed count |
| GRR | n(e^ε+d−2)/(e^ε−1)² |
| SUE | n e^(ε/2)/(e^(ε/2)−1)²; also exact for fixed data |
| OUE | 4n e^ε/(e^ε−1)² |

Variance measures squared error scale; standard deviation is its square root. Thus variance of order n/ε² corresponds to standard deviation of order √n/ε. OUE's improvement over SUE is not a universal factor of four; it depends on ε and the frequency regime.

## 3. Cryptographic foundations

### Symmetric encryption

Encryption and decryption share a secret key. Kerckhoffs's principle says security should depend on the key remaining secret, rather than the algorithm being secret.

- **Ciphertext-only attacker:** sees encrypted messages.
- **Known-plaintext attacker:** also knows corresponding plaintext/ciphertext pairs.
- **Confusion:** complicates the relationship between key and ciphertext.
- **Diffusion:** spreads an input change across many output bits.
- **Avalanche effect:** ideally, changing one input bit changes about half the output bits.

A block cipher processes fixed-length blocks. A stream cipher combines a message with a keystream, often C=M⊕K and M=C⊕K. Reusing the same keystream gives C₁⊕C₂=M₁⊕M₂, exposing a relation between plaintexts. A one-time pad additionally requires a truly random key as long as the message, used only once.

| Mode | Equation | Main point |
|---|---|---|
| ECB | Cᵢ=Eₖ(Mᵢ) | Equal plaintext blocks give equal ciphertext blocks |
| CBC encryption | Cᵢ=Eₖ(Mᵢ⊕Cᵢ₋₁), C₀=IV | Previous ciphertext changes the block-cipher input |
| CBC decryption | Mᵢ=Dₖ(Cᵢ)⊕Cᵢ₋₁ | Decrypt, then XOR with previous ciphertext |

CBC needs an appropriately generated IV; for its standard confidentiality guarantee the IV should be unpredictable. Plain encryption, including CBC, does not by itself authenticate data. The slide claiming symmetric encryption inherently authenticates users/data is too broad: a MAC or authenticated-encryption construction is needed.

Lecture parameters: DES has 64-bit blocks, 56 effective key bits, and 16 Feistel rounds. AES has 128-bit blocks and 128/192/256-bit keys. Treat historical deployment and hardware timing statements in the slides as historical context, rather than current guidance.

For N users with a distinct shared key for every pair, the number of keys is N(N−1)/2. A key distribution center reduces pairwise provisioning but becomes a trust and availability dependency.

### Hashes, MACs, and signatures

A cryptographic hash maps arbitrary-length messages to fixed-length digests. Collisions necessarily exist because the output space is finite; collision resistance means finding them is computationally difficult.

| Property | Attacker's task |
|---|---|
| Preimage resistance | Given digest y, find x with H(x)=y |
| Second-preimage resistance | Given x, find x′≠x with the same digest |
| Collision resistance | Find any two distinct messages with equal digests |

An unkeyed hash does not authenticate a sender: an attacker able to replace both message and digest can recompute it. A MAC uses a shared secret and provides integrity/authentication. A digital signature uses a private signing key and a public verification key. A signature alone does not hide the message.

### Birthday attack

For q independently uniform outputs among N possibilities:

\[
P(\text{collision})=1-\prod_{i=0}^{q-1}(1-i/N)
\approx1-e^{-q(q-1)/(2N)}.
\]

The union bound gives P≤q(q−1)/(2N). A b-bit ideal hash has N=2ᵇ outputs, giving generic collision work about 2^(b/2), versus preimage work about 2ᵇ. Thus a 256-bit ideal digest offers roughly 128-bit generic collision strength.

Do not confuse finding any collision with finding a second preimage for one prescribed message.

### Bloom filters

A standard Bloom filter has M initially zero bits and k hash functions. Insert an element by setting its k positions to 1. Query by checking those positions:

- At least one 0: definitely absent.
- All 1s: possibly present; hash collisions may cause a false positive.

There are no false negatives for correctly inserted items when bits are not cleared or corrupted. Deleting by clearing bits is unsafe because other elements may share them. A counting Bloom filter keeps counters to support deletion under its own assumptions.

After t insertions, under the random-hash approximation:

\[
P(\text{bit remains 0})=(1-1/M)^{kt}\approx e^{-kt/M},
\quad f\approx(1-e^{-kt/M})^k,
\quad k_{opt}=\frac Mt\ln2,
\quad f_{opt}\approx(0.6185)^{M/t}.
\]

At the optimum, about half the bits remain zero. Choose a nearby integer k. Example: 10 bits/item gives k≈6.93, so use about 7 hashes and false-positive probability about 0.82%.

### Modular arithmetic and number theory

The extended Euclidean algorithm finds u,v with au+bv=gcd(a,b). If the gcd is 1, u is a⁻¹ mod b.

Example: 60×5+13×(−23)=1, so 13⁻¹ mod 60=−23 mod 60=37. Modular division means multiplication by an inverse, not ordinary real-number division.

\[
\phi(n)=n\prod_{p\mid n}(1-1/p),\qquad
\phi(p)=p-1,\qquad\phi(pq)=(p-1)(q-1)
\]

for distinct primes p,q. Fermat: a^(p−1)≡1 mod p if p is prime and p∤a. Euler: a^φ(n)≡1 mod n if gcd(a,n)=1. Check these conditions before reducing exponents.

Example: φ(10)=4 and gcd(3,10)=1, so 3¹⁰⁴=(3⁴)²⁶≡1 mod 10.

CRT combines aᵢ residues modulo pairwise coprime mᵢ. With M=∏mᵢ and Mᵢ=M/mᵢ:

\[
x\equiv\sum_i a_iM_i(M_i^{-1}\bmod m_i)\pmod M.
\]

The answer is unique modulo M. Example: x≡1 mod 3, x≡2 mod 5, x≡3 mod 7 gives x≡52 mod 105. CRT also accelerates RSA decryption by computing separately modulo p and q and recombining.

### RSA and the common-modulus attack

Choose distinct primes p,q; n=pq; choose e coprime to φ(n); compute d=e⁻¹ mod φ(n). Public key: (n,e). Private exponent: d.

\[
c=m^e\bmod n,\qquad m=c^d\bmod n.
\]

Lecture example: p=7, q=11, n=77, φ=60, e=13, d=37. Message 15 encrypts to 64 and decrypts to 15. Textbook signature algebra is σ=H(m)^d mod n, verified by σ^e≡H(m) mod n; secure practical encodings require more than this bare expression.

If the same message is encrypted under the same modulus with coprime e₁,e₂, find x,y such that e₁x+e₂y=1. Then

\[
c_1^xc_2^y\equiv m^{e_1x+e_2y}\equiv m\pmod n.
\]

Negative powers use modular inverses. The usual derivation assumes gcd(m,n)=1; noninvertibility can instead reveal a factor through a gcd. This is an extended-Euclid attack, not the different-modulus CRT broadcast attack.

### Discrete logarithms, ElGamal, and Diffie–Hellman

A generator's powers cover its cyclic group. Computing y=gˣ is easy; recovering x from y is the discrete-logarithm problem. Exponents are considered modulo the group's order, which is p−1 for the full multiplicative group modulo prime p, or q in an order-q subgroup.

ElGamal public key: y=gˣ, secret x. For fresh random r:

\[
(u,v)=(g^r,my^r),\qquad m=v(u^x)^{-1}.
\]

Example modulo 13: g=2, x=5, y=6, m=7, r=4 gives u=3, v=11. Since uˣ=9 and 9⁻¹=3 mod 13, m=11×3 mod 13=7.

Diffie–Hellman exchanges gᵃ and gᵇ and derives K=gᵃᵇ. Its basic form does not authenticate peers; an active attacker can establish a separate key with each party. Signing and verifying identities and exchange values addresses this man-in-the-middle threat. DLP, computational DH, and decisional DH are related but distinct assumptions; DLP hardness alone is not a proof of every stronger security claim.

## 4. Homomorphic cryptography

Homomorphic encryption permits selected computations on ciphertexts without revealing plaintexts to the computing party.

| Scheme | Ciphertext operation | Plaintext result |
|---|---|---|
| Textbook RSA | Multiply ciphertexts | Multiply messages |
| ElGamal | Multiply matching ciphertext components | Multiply messages |
| Paillier | Multiply ciphertexts modulo n² | Add messages modulo n |
| Paillier | Raise ciphertext to known scalar k | Multiply message by k |
| BGN | Group multiplication; one pairing layer | Additions and one multiplication depth |
| FHE concept | Supported evaluation operations | General circuits with the scheme's required machinery |

These textbook homomorphic identities need not survive arbitrary encryption padding.

### Paillier: formulas and intuition

Choose p,q, n=pq, λ=lcm(p−1,q−1), L(u)=(u−1)/n, and valid g such that μ=[L(g^λ mod n²)]⁻¹ mod n exists.

\[
pk=(n,g),\quad sk=(\lambda,\mu),\quad
E(m;r)=g^m r^n\bmod n^2,\quad r\in\mathbb Z_n^*,
\]

\[
\operatorname{Dec}(c)=L(c^\lambda\bmod n^2)\mu\bmod n.
\]

Multiply ciphertexts:

\[
E(a;r)E(b;s)=g^{a+b}(rs)^n=E(a+b;rs)\pmod{n^2}.
\]

Raise to known k: E(a;r)^k=E(ka;rᵏ). This gives encrypted weighted sums ∏E(aᵢ)^bᵢ=E(Σaᵢbᵢ) when the computing party knows weights bᵢ. It does not generally multiply two unknown encrypted messages.

For g=n+1, the binomial theorem gives (1+n)^m≡1+mn mod n² because higher terms contain n². Fresh r makes encryption randomized. Ciphertexts operate modulo n²; messages operate modulo n.

### BGN and reduction reasoning

BGN uses composite-order groups and a bilinear pairing e(gᵃ,gᵇ)=e(g,g)ᵃᵇ. Encryption has form c=gᵐhʳ with h in a subgroup of order q. Raising c to q removes hʳ. Recovering m involves a discrete logarithm in a suitably small message range.

Pairing moves a product into a target group. Additions remain possible there, but the original pairing cannot simply be repeated on target-group ciphertexts. “One multiplication” means one multiplication depth; a sum of many pairwise products can still be evaluated.

Paillier's lecture security assumption is decisional composite residuosity: distinguishing n-th residues modulo n² from random group elements is hard. BGN uses a subgroup-decision assumption. Reduction template: assume an attacker distinguishes message encryptions, use it to distinguish hard-problem instances, and obtain a contradiction. IND-CPA means chosen-plaintext indistinguishability; it does not imply immunity to ciphertext manipulation.

Application: an encrypted sum permits an average after authorized aggregate decryption. Variance additionally needs Σxᵢ², since Var(X)=E[X²]−E[X]². Users can separately encrypt squared values, or a suitable multiplication-capable scheme can help. Whoever can decrypt individual submissions could learn them; aggregation needs an appropriate key/decryption policy.

## 5. Anonymous communication

### Anonymity goals

Sender anonymity hides the sender of a message. Recipient anonymity hides its recipient. Relationship anonymity hides which sender communicates with which recipient, even if participation is observable. Encrypting content does not automatically hide timing, volume, or endpoints.

### Mix networks and onion routing

A mix batches/shuffles messages and changes their representation to make input/output linking difficult. ElGamal re-encryption changes ciphertext randomness without changing the message:

\[
(u,v)=(g^r,my^r)\longmapsto(ug^s,vy^s)
=(g^{r+s},my^{r+s}).
\]

Ordinary re-encryption needs the recipient's public key y. Universal re-encryption attaches an encryption of 1. In the lecture's component order, (X,Y,W,Z)=(AʳM,gʳ,Aˢ,gˢ) transforms to (XWᵗ,YZᵗ,Wᵘ,Zᵘ), preserving the encrypted message without requiring the mixer to know A. Appropriate group/randomness conditions apply. Recipients test candidate outputs and reject ciphertexts not intended for their key.

Onion routing wraps messages in layers. Each router removes a layer and learns its adjacent hops. The entry can know the client; the exit can know the destination. The exit sees application plaintext only if it lacks end-to-end encryption. Anonymity depends on observation and collusion assumptions.

### Traffic analysis

| Attack | Information exploited |
|---|---|
| Communication-pattern correlation | Matching send/receive timing |
| Packet counting | Distinctive message counts or volume |
| Intersection | Repeatedly intersecting active-user or recipient sets |

Example intersection: possible peers {A,B,C}, then {B,C,D}, then {B,E}; their intersection is {B}. Randomizing ciphertexts alone does not remove these metadata signals.

### DC-nets and k-anonymous message transmission

Party i splits message Mᵢ into additive shares sᵢ,₁,…,sᵢ,ₙ with Σⱼsᵢ,ⱼ=Mᵢ modulo the message modulus. Non-senders use Mᵢ=0. Each party j broadcasts Bⱼ=Σᵢsᵢ,ⱼ. Then

\[
\sum_jB_j=\sum_iM_i.
\]

If exactly one party sends, the sum reveals the message while the sharing arrangement hides its source under the protocol's assumptions. Multiple simultaneous senders can collide. The basic construction costs O(n²) communication and can be jammed by dishonest broadcasts.

k-AMT uses smaller DC-net groups to provide an anonymity set of k rather than the entire network. Verifiable secret sharing and commitments help detect inconsistent shares. They do not automatically solve a party intentionally transmitting every turn and monopolizing the channel.

### Pedersen commitment

In a prime-order group, with g,h generators and the committer not knowing log_g h:

\[
C(x;r)=g^xh^r.
\]

Open by revealing x,r. It is perfectly hiding with uniform r and computationally binding under the discrete-log assumption. If h=gᵃ and two different openings satisfy gˣhʳ=gˣ′hʳ′, then

\[
a=(x'-x)(r-r')^{-1}\pmod q.
\]

This would reveal the supposedly unknown discrete log. The setup's trapdoor must not be known to a party expected to be bound. Commitments also combine: C(x;r)C(y;s)=C(x+y;r+s).

## 6. Private information retrieval (PIR)

PIR lets a client retrieve xᵢ while hiding **index i** from the database server. The server already knows its database values; privacy concerns which one was selected.

Downloading the entire database gives trivial information-theoretic query privacy but linear communication. In the basic single-server model in the lecture, perfect information-theoretic privacy cannot beat this linear barrier. Two routes are noncolluding replicated servers or computational assumptions.

### Two-server XOR protocol

Choose uniform Q₁∈{0,1}ⁿ and set Q₂=Q₁⊕eᵢ, where eᵢ has one 1 at position i. Server j replies aⱼ=⊕ₗ(Qⱼ,ₗxₗ). Thus

\[
a_1\oplus a_2=\bigoplus_\ell[(Q_{1,\ell}\oplus Q_{2,\ell})x_\ell]=x_i.
\]

Each server separately sees a uniform query. If they collude, Q₁⊕Q₂=eᵢ reveals i. Basic communication is two n-bit queries and two one-bit responses: O(n), despite the short answers.

### Matrix and cube improvements taught in the lecture

Arrange n bits in a roughly √n×√n matrix. Send two masked column-selection vectors of length √n; each server returns one parity per row. Combining replies gives the selected column, from which the client takes the desired row. Queries and replies have O(√n) total size.

For the cube construction, let n=m³ and the target be (i,j,k). Choose independent random vectors a,b,c∈{0,1}ᵐ. Server 2 receives a⊕eᵢ, b⊕eⱼ, c⊕eₖ. Each server returns three length-m arrays: parity sums over its selected y,z positions for every x, over x,z for every y, and over x,y for every z.

The client combines the two x-array entries at i, the two y-array entries at j, and the two z-array entries at k. Over XOR arithmetic, paired unwanted terms cancel and the target occurs three times, leaving its value. Each direction communicates 6m bits, hence O(n^(1/3)). These are the constructions/bounds discussed in the slides, not a claim about the latest best bounds.

### Computational PIR with homomorphic encryption

For a matrix, the client encrypts a one-hot column selector s, where sᵢ=1 and other entries are 0. For each row j, the server computes

\[
C_j=\prod_k E(s_k)^{x_{j,k}}=E\left(\sum_k s_kx_{j,k}\right)=E(x_{j,i}).
\]

The client decrypts the selected column and keeps the desired row. This works for suitably encoded integer entries too, with modular wraparound controlled. Communication is O(√n) ciphertexts, rather than O(√n) literal bits; ciphertext size depends on the security parameter. Server work can still scan the database. PIR does not normally guarantee that the client learns only one item.

## 7. Oblivious transfer (OT)

In 1-out-of-2 OT, the sender holds m₀,m₁ and the receiver chooses σ∈{0,1}. The receiver learns mσ, the sender does not learn σ, and the receiver learns nothing beyond the allowed selected result about the other message. 1-out-of-n generalizes the choice.

| Guarantee | PIR | OT |
|---|---|---|
| Sender/server does not learn selected index | Yes | Yes |
| Receiver restricted to one selected item | Not required by ordinary PIR | Yes, within the security model |

### RSA-based lecture protocol

The sender publishes N,e and random r₀,r₁, keeping d. Receiver chooses random k and sends v=rσ+kᵉ mod N. Sender computes kᵢ=(v−rᵢ)ᵈ mod N and returns m′ᵢ=mᵢ+kᵢ mod N. The receiver gets mσ=m′σ−k mod N because kσ=k. The slides label this an honest-but-curious construction; do not assume malicious security without validation and a proof.

### Paillier-based lecture protocol

Sender holds the Paillier secret key and sends E(rᵢ) for random masks rᵢ. Receiver returns c=E(rσ)E(w) for fresh random w. Sender decrypts c/E(rᵢ), obtaining Cᵢ=rσ+w−rᵢ, then returns zᵢ=mᵢ+Cᵢ. For the selected index, zσ−w=mσ. All plaintext arithmetic is modulo the message modulus. This explains the correctness algebra; robust privacy depends on the full protocol assumptions, fresh randomization, and allowed queries.

### DDH-based construction: key derivation

The receiver prepares public group elements gᵃ,gᵇ,gᶜ⁰,gᶜ¹, with cσ=ab and the other exponent chosen differently. The sender chooses random r,s and computes

\[
w=g^{ar+s},\qquad \pi_i=g^{c_ir+bs}.
\]

For the selected index:

\[
\pi_\sigma=g^{abr+bs}=w^b.
\]

The receiver can therefore recover its chosen mask. DDH hides which tuple has the DH relation. For the other index, the distinct linear combination remains masked under the construction's assumptions. Encoding a group element as an arbitrary-length XOR pad needs an appropriate derivation/encoding step; the slide's XOR expression is schematic. Validating group elements and preventing duplicate DH slots matter.

### Why encrypted selection alone is insufficient for OT

A client can send E(1) at its chosen index and E(0) elsewhere. A server returns ∏E(sᵢ)^mᵢ=E(Σsᵢmᵢ). An honest client learns one message, but a malicious client can encrypt multiple 1s, learning a sum or other unauthorized combination. The protocol must enforce a valid one-hot selection and prevent other leakage. Correctness for honest participants is not a complete security proof.

## 8. Zero-knowledge proofs

A prover convinces a verifier of a statement or knowledge of a witness without revealing additional information beyond the allowed statement.

| Property | Meaning |
|---|---|
| Completeness | An honest prover with a valid witness is accepted |
| Soundness | A cheating prover cannot establish a false statement except with bounded probability |
| Zero knowledge | A simulator without the witness can reproduce the verifier's view |

The verifier's view includes received messages and its randomness. The simulator formulation is more precise than saying the verifier's view “never changes”: it learns that the statement holds, but no extra witness information.

### Cave and graph-isomorphism examples

In the cave example, the prover commits to entering a branch before the verifier randomly requests an exit side. Knowing the door secret permits either response. Without it, success is at most 1/2 per independent round. After k rounds, cheating probability is at most 2⁻ᵏ. For k=10 this is 1/1024.

For graph isomorphism, the prover knows φ:G→H. It sends a randomly relabeled copy C=π(G). The verifier requests an isomorphism from G or H to C. The prover answers π for G, or π∘φ⁻¹ for H. If G,H are not isomorphic, C cannot match both, giving the same 1/2 soundness error per round. A simulator guesses the requested graph, builds an appropriate random copy, and retries/rewinds when its guess is wrong. This creates the proper transcript distribution without the secret isomorphism.

### Schnorr identification

Use an order-q group generated by g. Secret ω, public x=g^ω:

1. Commitment: choose fresh r, send a=gʳ.
2. Challenge: verifier samples c∈Zq.
3. Response: send z=r+cω mod q.
4. Verify gᶻ=a xᶜ.

Correctness follows from g^(r+cω)=gʳ(g^ω)ᶜ. The secret is ω; x is public, despite a wording slip in the slide.

If the same commitment/randomness is reused with challenges c≠c′, then

\[
\omega=(z-z')(c-c')^{-1}\pmod q.
\]

Fresh randomness is essential. A simulator can choose c,z and set a=gᶻx⁻ᶜ to form a valid transcript for an honest random challenge. This is the basic honest-verifier simulation intuition; stronger verifier models require appropriate qualifications.

### Noninteractive form

Fiat–Shamir replaces the verifier's challenge with a hash of the statement/context and commitment. The slide's signature form uses c=H(m∥a); fuller formulations bind the public key and other context as appropriate. The prover publishes the response and enough information to verify the hash challenge. This is not a claim that any interactive proof becomes secure merely by inserting a hash; the transformation has assumptions.

## 9. Private matching in mobile social networks

Mobile matching adds proximity and interests to the privacy problem. The intended result might be equality, membership, intersection, its size, or only whether a threshold is met. These outputs leak different amounts.

### Encrypted common-interest count

Represent interests by binary vectors a,b. Their dot product counts common interests:

\[
a\cdot b=\sum_i a_ib_i.
\]

Alice sends cᵢ=E(aᵢ). Bob computes D=∏cᵢ^bᵢ=E(Σaᵢbᵢ). Alice decrypts the count and compares it with threshold t. The slides' example has four common interests, so t=4 passes and t=5 fails.

**Privacy limit:** Alice learns the exact count, including when below threshold. Therefore this alone does not implement the stronger goal of revealing only whether the threshold passes. Malicious inputs or repeated specially chosen queries can reveal more. A secure comparison and additional input/security controls may be needed for the stronger functionality.

### Equality testing

Sending H(y) lets Alice compare it with H(x), but a small input domain permits offline guessing: hash every possible diagnosis, postcode, or interest and compare. Hashing is not equivalent to hiding a low-entropy secret.

In the OT-based example, Bob chooses two independent λ-bit labels Kᵢ,₀,Kᵢ,₁ for each bit position. Alice uses OT with choice xᵢ to receive Kᵢ,ₓᵢ. Bob sends T=⊕ᵢKᵢ,ᵧᵢ. Alice computes T′=⊕ᵢKᵢ,ₓᵢ and checks equality. If x=y, they match. For a fixed unequal pair in this basic construction, hidden random labels make accidental equality probability 2⁻λ. Full security still depends on the adversary model and OT construction.

### Inclusion, intersection, and hashing

| Functionality | Inputs | Intended result |
|---|---|---|
| Equality | x,y | Whether x=y |
| Set inclusion/membership | x,Y | Whether x∈Y |
| Set intersection (PSI) | X,Y | X∩Y |
| Intersection cardinality | X,Y | |X∩Y| |
| Threshold matching | X,Y,t | Whether |X∩Y|≥t |

Membership can run equality tests against every y∈Y. Naive intersection of two size-n sets needs n² pairwise comparisons. Hashing both sets into m bins means comparing only within matching bins. With balanced occupancy n/m per side, comparison work is approximately

\[
m(n/m)^2=n^2/m.
\]

Choosing m≈n/log n gives O(n log n) comparisons in the lecture's analysis. This is a balanced-bin estimate, not an unconditional worst-case guarantee; collisions, overloaded bins, and padding must be handled. Revealing each equality result may also reveal which position matches, which is more than a membership bit if the intended output is narrower.

## 10. Sample quiz: worked answer map

| Q | Answer | Reason to remember |
|---|---|---|
| 1 | B | Paillier ciphertext multiplication adds plaintexts modulo n |
| 2 | B | Extended Euclid gives e₁x+e₂y=1, so c₁ˣc₂ʸ=m |
| 3 | A | 3⁴≡1 mod 10 and 104 is divisible by 4 |
| 4 | A | Clearing a shared Bloom-filter bit can introduce false negatives |
| 5 | A | LDP randomizes individual inputs; its count error typically scales as √n |
| 6 | C | Larger sensitivity increases Laplace scale and moves geometric α toward 1 |
| 7 | C | Candidate (1,0,1,0) satisfies both error constraints |

### Q7: dot products and elimination

For c=(c₁,c₂,c₃,c₄), q₁=(0,1,1,0) selects positions 2 and 3, so c·q₁=c₂+c₃. Query q₂=(1,0,1,1) selects positions 1,3,4, so c·q₂=c₁+c₃+c₄. The dots mean multiply corresponding entries and sum, not concatenate symbols.

With responses 1.5 and 2.5 and error bound 0.5, a surviving candidate needs c·q₁∈[1,2] and c·q₂∈[2,3].

| Candidate | First sum | First error | Second sum | Second error | Survives? |
|---|---:|---:|---:|---:|---|
| (0,0,0,0) | 0 | 1.5 | 0 | 2.5 | No |
| (1,0,0,1) | 0 | 1.5 | 2 | 0.5 | No |
| (1,0,1,0) | 1 | 0.5 | 2 | 0.5 | Yes |
| (0,1,0,0) | 1 | 0.5 | 0 | 2.5 | No |

The bound is inclusive: error exactly 0.5 is allowed. Surviving these queries means consistent with the observations, not necessarily uniquely determined among all 16 possible binary databases.

## 11. Exam traps and final revision checklist

1. **Operation level:** distinguish ciphertext multiplication from plaintext multiplication.
2. **Modulus:** Paillier encrypts modulo n² and represents messages modulo n.
3. **Inverse:** modular division requires invertibility; negative exponents mean inverses.
4. **Adjacency:** histogram sensitivity is 1 for add/remove and up to 2 for replacement.
5. **DP parameters:** smaller ε means more privacy and usually more noise.
6. **Geometric convention:** here α>1; closer to 1 means more noise.
7. **Composition:** sequential adds; disjoint-person parallel uses maximum under the applicable adjacency assumptions.
8. **LDP variance:** independent reports need not be identically distributed; specify fixed-data or population sampling.
9. **Bloom filter:** all 1s means “possibly”; a 0 means “definitely absent.”
10. **Anonymity:** ciphertext secrecy does not hide timing, message counts, or endpoints.
11. **PIR versus OT:** PIR hides the index; OT additionally restricts the receiver's information.
12. **ZKP:** completeness, soundness, and zero knowledge are three separate properties.
13. **Protocol security:** deriving the intended answer for an honest client does not establish malicious security.
14. **Allowed output:** count, intersection, and threshold bit are different disclosures.

### Questions to answer aloud

- Why does removing names fail against linking attacks?
- Give a 3-anonymous class that still reveals everyone's diagnosis.
- Derive the Laplace privacy-ratio bound using the reverse triangle inequality.
- Derive the randomized-response estimator from its expectation.
- Explain why changing a one-hot value changes two coordinates.
- Derive the Bloom-filter false-positive approximation and optimal k.
- Recover a modular inverse using extended Euclid.
- Explain RSA's common-modulus attack without factoring n.
- Derive Paillier's encrypted weighted-sum identity.
- Explain what each Tor hop learns and what traffic analysis can still reveal.
- Show why the two-server XOR PIR answers combine to xᵢ.
- Explain how a malicious encrypted selector can defeat a naive OT design.
- Derive Schnorr's verification equation and the nonce-reuse extraction formula.
- Explain why a common-interest count leaks more than a threshold decision.

## Source guide

| Supplied file | Main sections used |
|---|---|
| Topic 01-w-updated.pdf | Security, anonymization, Incognito, DP, sensitivity, noise, composition |
| Topic 02-w.pdf | Big data, LDP, frequency oracles, BRR/GRR/SUE/OUE |
| Topic 03-w.pdf | Symmetric encryption, hashes, birthday bound, Bloom filters, number theory, RSA, ElGamal, DH |
| Topic 04 -w.pdf | Homomorphic encryption, Paillier, BGN, security reductions |
| Topic 05-w.pdf | Mix networks, onion routing, universal re-encryption, DC-nets, commitments |
| Topic 06-w.pdf | Information-theoretic and computational PIR constructions |
| Topic 07-w.pdf | RSA-, Paillier-, and DDH-based OT; insecure selector example |
| Topic 08-w.pdf | Cave, graph isomorphism, simulation, Schnorr identification/signature |
| Topic 09-w.pdf | Friend matching, equality, set inclusion, PSI, hashing |
| CISC-820-Sample-Quiz.pdf | Seven worked multiple-choice questions |

Clarifications in these notes identify mathematical assumptions or limitations rather than reproduce every informal slide statement verbatim. The coverage is based on the supplied materials, not a prediction of the actual exam questions.
