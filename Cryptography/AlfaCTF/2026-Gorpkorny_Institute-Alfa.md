# GorpKor Institute — Writeup

## Challenge

The website accepts a YAML certificate (`attestat`) and checks whether the applicant has an average grade of `5.00`.

The goal is to submit a valid certificate for passport number `1337676769`, with all grades changed to `5`, while still passing the certificate signature check.

Flag:

```text
alfa{attES7A7_V_krOVI_po_B0kAm_koNVoy_4_M3Ny4_VeSu7_p0d_S1reni_v0Y}
```

---

## Initial Observation

The challenge description says:

- Certificates are YAML files.
- Each certificate is linked to a passport number.
- The certificate hash is stored in the register.
- We need an average score of `5`.

Submitting a modified certificate with all grades set to `5` passed most checks:

```json
{
  "label": "Средний балл",
  "value": "5.00",
  "ok": true
}
```

However, the signature failed:

```text
Подпись: неверна
ожидалось 0x00c0f995c413ce93
```

Translation:

```text
Файл         = File
Паспорт      = Passport
Структура    = Structure
Средний балл = Average score
Подпись      = Signature
```

So the YAML structure and grades were accepted, but the raw file hash did not match the stored signature.

---

## Vulnerability

From the provided source code, the signature function was a weak polynomial hash:

```go
func sigOf(b []byte) uint64 {
    var h uint64
    for _, c := range b {
        h = (h*257 + uint64(c)) % ((1 << 56) - 5)
    }
    return h
}
```

The important issue is that the signature is calculated over the raw YAML bytes.

YAML supports comments using `#`, and comments do not affect the parsed data. This means we can append a comment to the YAML file:

```yaml
# anything_here
```

The server will still parse the same passport and grades, but the raw bytes used for the hash will change.

Because the hash is weak and linear, we can brute-force or solve for a short comment suffix that makes the modified YAML collide with the expected signature.

---

## Exploit Idea

We create a YAML certificate with:

- the correct passport number
- all grades changed to `5`
- a specially chosen YAML comment at the end

Final payload:

```yaml
паспорт: "1337676769"
оценки:
  физика: 5
  химия: 5
  информатика: 5
  английский: 5
  русский: 5
  литература: 5
  физкультура: 5
  обществознание: 5
  алгебра: 5
  история: 5
  география: 5
  биология: 5
  геометрия: 5
# IQQux}?d*1n3Jx
```

The final line is only a YAML comment, so it does not change the parsed certificate. But it changes the raw bytes so that the signature becomes:

```text
0x00c0f995c413ce93
```

---

## Exploit Commands

Create the payload file:

```bash
cat > payload.yml <<'EOF'
паспорт: "1337676769"
оценки:
  физика: 5
  химия: 5
  информатика: 5
  английский: 5
  русский: 5
  литература: 5
  физкультура: 5
  обществознание: 5
  алгебра: 5
  история: 5
  география: 5
  биология: 5
  геометрия: 5
# IQQux}?d*1n3Jx
EOF
```

Submit it:

```bash
base='https://gradebook-3nsywsx2.alfactf.ru'

curl -k -s "$base/submit" \
  -X POST \
  --data-urlencode attestat@payload.yml | jq
```

---

## Result

The server accepted the forged certificate:

```json
{
  "label": "Подпись",
  "value": "верна (0x00c0f995c413ce93)",
  "ok": true
}
```

Then it returned the flag:

```json
{
  "flag": "alfa{attES7A7_V_krOVI_po_B0kAm_koNVoy_4_M3Ny4_VeSu7_p0d_S1reni_v0Y}"
}
```

---

## Flag

```text
alfa{attES7A7_V_krOVI_po_B0kAm_koNVoy_4_M3Ny4_VeSu7_p0d_S1reni_v0Y}
```

---
* [🔙 Back to Cryptography Directory](../)
* [🔙 Back to Cryptography Index Directory](../INDEX.md)
* [🔙 Back to Main Directory](../../README.md)
