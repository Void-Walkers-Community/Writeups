Flagpression — GlacierCTF

The challenge provided a very precise decimal number and a photograph of handwritten notes. The goal was to understand how the number represented a hidden flag.

The photograph contained the main clues: an ordered alphabet called \(X\), a uniform probability formula \(p(x)=1/|X|\), and a diagram showing an interval being repeatedly subdivided. Together with the challenge name, these pointed toward arithmetic coding, a compression technique that represents a message as a number inside an interval.

Carefully transcribing the handwritten alphabet gave 33 symbols, in this order:

\(g,c,t,f,\{,n,0,T,?,h,a,r,8,\_,p,Y,w,m,s,3,9,1,A,4,E,2,y,6,Z,S,B,7,\}\)

The outer braces on the paper enclosed the set, while the inner opening and closing braces were actual alphabet characters. Several handwritten symbols were ambiguous, particularly zero versus uppercase O, nine versus lowercase g, and uppercase S versus five. These distinctions mattered because changing the alphabet would change the recovered message.

Arithmetic coding starts with the interval \([0,1)\). Each character selects a smaller subinterval according to its probability and position in the alphabet. Repeating this process produces a final interval representing the entire message. Any suitable number inside that interval can act as its encoded representation.

The first attempt used exact, equal subdivisions of size \(1/33\). Decoding recovered the expected gctf{ prefix, which strongly supported the arithmetic-coding hypothesis. However, the remaining characters became meaningless, and the resulting candidate was rejected.

The missing detail was how the encoder calculated its probability boundaries. A matching model emerged when the boundaries were constructed by repeatedly adding the ordinary floating-point value of \(1/33\). Floating-point rounding meant these boundaries differed slightly from the mathematically exact fractions. After 33 additions, the final boundary was approximately \(0.9999999999999993\).

Although these errors were tiny, repeated decoding magnified them. This explained why the exact model initially recovered the correct prefix before diverging.

Using the accumulated boundaries, each decoding step located the subinterval containing the current number, emitted its character, and rescaled that subinterval back to the full range. The supplied decimal was preserved with exact rational arithmetic throughout decoding. This recovered a coherent, 64-character flag.

Verification went beyond checking whether the output looked plausible. Re-encoding the recovered message with the same boundary table, using 200-digit decimal precision and selecting the final interval’s midpoint, reproduced every digit of the supplied decimal. The flag was then submitted and accepted.

The challenge demonstrated that compression provides no secrecy when its model is known—and that reproducing numerical implementation details can be essential when reversing an encoding.

Flag: gctf{876aB98a_c0mpr3ss0rS_4r3_Aw3s0mE_Ar4nT_Th3y_???_12Yha76B2Z}

