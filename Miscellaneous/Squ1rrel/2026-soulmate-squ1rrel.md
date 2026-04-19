misc/soulmate was solved by reading the backend source and noticing the real gate is /submit-u, not the birthday UI. The service loads an 8D PCA basis (/health leaks the bounds), clips the user-supplied latent u, maps it into StyleGAN w, then runs a celebrity classifier and only returns the flag when the Tom Cruise probability exceeds 0.15. I queried the endpoint directly and brute-forced the 256 sign combinations at the PCA extremes, which found a winning vector ([-,+,+,-,+,-,+,+], mask 214) that pushed tom_score to 0.2036 and revealed squ1rrel{7h3_church_c0n6r47ul4735_mr5_cru153!!}.
---
* [🔙 Back to Miscallaneous Directory](../)
* [🔙 Back to Miscellaneous Index Directory](../INDEX.md)
* [🔙 Back to Main Directory](../../README.md)
