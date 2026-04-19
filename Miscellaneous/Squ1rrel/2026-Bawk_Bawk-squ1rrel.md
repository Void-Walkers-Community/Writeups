The challenge was a 64×64 RGBA image-classification CTF with a PyTorch checker model and a chicken image. I first inspected the model archive and reconstructed the network locally, then verified the original image was classified as plush_chicken. Since the site explicitly hinted that “even if you change one pixel” mattered, I brute-forced single-pixel RGB corner changes offline against the local model until I found that changing pixel (55, 7) to white flipped the prediction to baby_chick. Submitting that adversarial PNG satisfied the checker and returned the flag image, which contained squ1rrel{banana_bunny_boat_tea}.
---
* [🔙 Back to Miscallaneous Directory](../)
* [🔙 Back to Miscellaneous Index Directory](../INDEX.md)
* [🔙 Back to Main Directory](../../README.md)
