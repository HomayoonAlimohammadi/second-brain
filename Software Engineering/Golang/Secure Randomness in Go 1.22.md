#golang #software_engineering #security
## References
https://go.dev/blog/chacha8rand
#### Introduction
The post discusses the evolution of random number generation in Go, especially focusing on the enhancements in Go 1.22 that integrate cryptographic techniques to improve the randomness quality of `math/rand`.

#### Statistical vs. Cryptographic Randomness
- **Statistical Randomness**: Sufficient for simulations, non-cryptographic algorithms, etc. Traditional generators like linear congruential generators (LCGs) were common but predictable.
- **Cryptographic Randomness**: Essential for security-dependent applications, requires unpredictability even when the generation method is known.

#### Historical Context
Early random number generators (RNGs) like those in Unix V3 were simplistic and predictable, using methods such as linear congruential generators. These methods suffer from sequence predictability and short cycles.

#### The Go 1 Generator
- Utilizes a linear-feedback shift register approach, inexpensive in computation but exposes internal state, making it predictable and insecure.

#### The PCG Generator
- Introduced in `math/rand/v2`, PCG (Permuted Congruential Generator) is a modern statistical RNG that improves upon the predictability of LCGs but still isn't suited for cryptographic purposes.

#### Cryptographic Randomness in Operating Systems
- OSs provide randomness from hardware sources, which is then extended via cryptographic methods. Go interfaces with these system capabilities through the `crypto/rand` package.

#### The ChaCha8Rand Generator
- Go 1.22 introduces `ChaCha8Rand`, a variant of the ChaCha stream cipher, optimized for speed and security. It combines robustness with efficient generation speeds, making it suitable for both cryptographic and non-cryptographic applications.
- It's used in the new `math/rand/v2` as well as improved `math/rand` when not explicitly seeded.

#### Security and Performance Implications
- By default, `math/rand` in Go 1.22 uses `ChaCha8Rand` if not seeded, which improves security without requiring changes to existing code.
- Performance comparisons show that while `ChaCha8Rand` is slightly slower than older methods, the security benefits outweigh the performance costs.

#### Conclusion
Go 1.22 aims to secure applications by default by addressing common errors in random number usage. The transition to using cryptographic techniques in general-purpose RNGs marks a significant step towards safer programming practices.


