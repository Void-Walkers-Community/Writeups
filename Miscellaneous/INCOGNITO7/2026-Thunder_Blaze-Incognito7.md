I used this to recover the flag 
```
#!/usr/bin/env python3
from pwn import *
import re, ctypes, time

context.log_level = 'info'

lib = ctypes.CDLL('./seq.so')
lib.compute.restype = ctypes.c_uint64
lib.compute.argtypes = [ctypes.c_uint64]*6

FONT = {
'0':'###\n#.#\n#.#\n#.#\n###',
'1':'..#\n..#\n..#\n..#\n..#',
'2':'###\n..#\n###\n#..\n###',
'3':'###\n..#\n###\n..#\n###',
'4':'#.#\n#.#\n###\n..#\n..#',
'5':'###\n#..\n###\n..#\n###',
'6':'###\n#..\n###\n#.#\n###',
'7':'###\n..#\n..#\n..#\n..#',
'8':'###\n#.#\n###\n#.#\n###',
'9':'###\n#.#\n###\n..#\n###',
}
FONT = {d: v.split('\n') for d,v in FONT.items()}
SOLID = set('#@%*')

def parse_number(block_text):
    rows = block_text.split('\n')
    # strip empty
    rows = [r for r in rows if r.strip() or r==rows[0]]
    # Should be 5 rows
    rows = rows[:5]
    w = max(len(r) for r in rows)
    ndig = (w + 2) // 5
    out = ''
    for d in range(ndig):
        start = d*5
        pat = []
        for r in rows:
            r2 = r.ljust(w)
            pat.append(''.join('#' if r2[start+c] in SOLID else '.' for c in range(3)))
        best=None; bd=99
        for dd,f in FONT.items():
            dist = sum(a!=b for x,y in zip(pat,f) for a,b in zip(x,y))
            if dist<bd:
                bd=dist; best=dd
        out += best
    return int(out)

def solve_task2(prompt):
    m = re.search(r'S_0 = (\d+)', prompt)
    S0 = int(m.group(1))
    S1 = int(re.search(r'S_1 = (\d+)', prompt).group(1))
    # Extract C, D, E blocks
    def extract(label):
        mm = re.search(r'--- VALUE OF '+label+r' ---\s*\n(.*?)(?=\n\n|\n---|\nFind)', prompt, re.S)
        return mm.group(1)
    C = parse_number(extract('C'))
    D = parse_number(extract('D'))
    E = parse_number(extract('E'))
    m = re.search(r'S_(\d+)', prompt.split('Find')[1])
    N = int(m.group(1))
    print(f"S0={S0} S1={S1} C={C} D={D} E={E} N={N}")
    return str(lib.compute(S0, S1, C, D, E, N))

def handle(prompt):
    if 'Calculate:' in prompt:
        m = re.search(r'Calculate:\s*(\d+)\s*([*+\-])\s*(\d+)', prompt)
        a,op,b = int(m.group(1)), m.group(2), int(m.group(3))
        return str({'*':a*b,'+':a+b,'-':a-b}[op])
    if 'Find the value of S_' in prompt:
        return solve_task2(prompt)
    return None

def main():
    r = remote('34.131.216.230', 1337)
    buf = ''
    while True:
        try:
            chunk = r.recv(timeout=3).decode(errors='replace')
        except EOFError:
            print("EOF"); print(buf); return
        if not chunk:
            break
        buf += chunk
        if buf.rstrip().endswith('>'):
            t=time.time()
            ans = handle(buf)
            print(f"handle: {time.time()-t:.3f}s  ans={ans!r}")
            if ans is None:
                print("UNKNOWN PROMPT:\n", buf)
                r.interactive()
                return
            r.sendline(ans.encode())
            buf = ''
    print(buf)

if __name__ == '__main__':
    main()
```
---
* [🔙 Back to Miscallaneous Directory](../)
* [🔙 Back to Miscellaneous Index Directory](../INDEX.md)
* [🔙 Back to Main Directory](../../README.md)
