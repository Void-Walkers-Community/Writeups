flow:

  1. Open the PCAP and inspect TCP streams to 31337.
  2. Ignore plaintext decoy streams that contain HTTP/markdown/prompt-injection content.
  3. Find a real encrypted session. The server banner reveals:
      - SUGAR_PROTOCOL v1.0
      - salt: a3f7c9b1e2d45608
      - cipher: AES-256-CBC
      - KDF: SHA256(PASSPHRASE||SALT)
  4. Recover the frame format from the valid stream:
      - 4-byte big-endian length
      - then payload
      - payload = IV (16 bytes) || AES-CBC ciphertext
  5. Brute/verify candidate passwords against the encrypted stream. The correct one is:
      - chocolate
  6. After decryption, the real session contains valid shell-like commands such as ls, pwd, whoami, proving the protocol reconstruction is correct.
  7. Connect to the live server, send encrypted commands, and read the flag with:
      - cat flag.txt

  Flag

  KubSTU{d0r4_dur4_sug4r_ch0c0l4t3_v1b3z}

  Solve script

  import socket
  from hashlib import sha256
  from os import urandom
  from cryptography.hazmat.primitives.ciphers import Cipher, algorithms, modes

  HOST = "217.26.29.80"   # or 62.113.108.12
  PORT = 31337

  PASSPHRASE = "chocolate"
  SALT = "a3f7c9b1e2d45608"
  KEY = sha256((PASSPHRASE + SALT).encode()).digest()

  def aes_encrypt(msg: bytes) -> bytes:
      pad = 16 - (len(msg) % 16)
      padded = msg + bytes([pad]) * pad

      iv = urandom(16)
      enc = Cipher(algorithms.AES(KEY), modes.CBC(iv)).encryptor()
      ct = enc.update(padded) + enc.finalize()

      payload = iv + ct
      return len(payload).to_bytes(4, "big") + payload

  def aes_decrypt(payload: bytes) -> bytes:
      iv = payload[:16]
      ct = payload[16:]

      dec = Cipher(algorithms.AES(KEY), modes.CBC(iv)).decryptor()
      pt = dec.update(ct) + dec.finalize()

      pad = pt[-1]
      return pt[:-pad]

  def recvn(sock: socket.socket, n: int) -> bytes:
      data = b""
      while len(data) < n:
          chunk = sock.recv(n - len(data))
          if not chunk:
              raise EOFError("socket closed")
          data += chunk
      return data

  def recv_line(sock: socket.socket) -> bytes:
      data = b""
      while not data.endswith(b"\n"):
          chunk = sock.recv(1)
          if not chunk:
              raise EOFError("socket closed")
          data += chunk
      return data

  def recv_frame(sock: socket.socket) -> bytes:
      hdr = recvn(sock, 4)
      size = int.from_bytes(hdr, "big")
      payload = recvn(sock, size)
      return aes_decrypt(payload)

  with socket.create_connection((HOST, PORT), timeout=5) as s:
      s.settimeout(5)

      # current server sends only these two plaintext banner lines
      print(recv_line(s).decode(errors="replace").strip())
      print(recv_line(s).decode(errors="replace").strip())

      for cmd in [b"ls", b"cat flag.txt"]:
          s.sendall(aes_encrypt(cmd))
          resp = recv_frame(s)
          print(f"$ {cmd.decode()}")
          print(resp.decode(errors="replace"))

---
* [🔙 Back to Miscallaneous Directory](../)
* [🔙 Back to Miscellaneous Index Directory](../INDEX.md)
* [🔙 Back to Main Directory](../../README.md)
