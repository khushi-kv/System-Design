# Checksums

Checksums explained in plain words, with no math and no jargon.

**TL;DR:** A checksum is a small number calculated from your data. If the data changes, the number changes. That is how computers notice something went wrong, even when nothing says “I’m broken.”

*Refer to the image here: ./diagrams/01-intro.svg*

## The problem

You download a big file. The progress bar hits 100%. You open it, and it won’t work.
Nothing crashed. No error appeared. A few bytes got damaged on the way, or the download stopped early, and the file looks perfectly normal from the outside.

This happens more than you would think. A network signal picks up noise. A disk returns the wrong block. A transfer gets cut off halfway. The data never announces any of this, so computers need a way to check for themselves.

---

## What is a checksum?

Think about a shop receipt. You buy five items and the bill says Total: ₹500. At home you add up the prices again and get ₹480. Something is off. You don’t know which item is wrong yet, but you know the receipt doesn’t match.

A checksum works the same way:
1. Take some data (a file, a message, a chunk on a disk).
2. Run it through a formula to get a short value. That is the checksum.
3. Later, run the same formula on the data you received.
4. Same value: the data probably didn’t change. Different value: something changed.

*Refer to the image here: ./diagrams/02-how-it-works.svg*

A checksum is tiny compared to the data. A file can be gigabytes in size, but its checksum might be just a few bytes.

---

## What a checksum can and can’t do

* **It can detect that data changed.** Flipped bits, half-finished downloads, damaged files, and wrong disk blocks all show up as a mismatch.
* **It can’t fix the data.** A checksum only says something is wrong. Fixing it takes something else: asking for the data again, reading a backup copy, or alerting a person.
* **It doesn’t hide anything.** A checksum is not encryption. Anyone can still read the data.
* **It doesn’t stop attackers.** If someone can change your file, they can also recalculate the checksum to match. A plain checksum protects against accidents, not against people trying to cheat.

That last point is why there is more than one kind of checksum.

---

## The four tools, in plain words

Different problems need different tools.

### 1. CRC: for accidents

CRC is fast and cheap. It is built to catch the kind of damage that happens naturally, like noise on a cable or a flipped bit on a disk. Networking and storage use it everywhere.

But a CRC is easy to fake on purpose, so don’t use it to defend against attackers.

### 2. SHA-256: a strong fingerprint

SHA-256 produces a longer, much stronger value that works like a fingerprint for your data. Change even one character and you get a completely different result.

You can try this yourself in two minutes. Open PowerShell and create a tiny file:

```powershell
echo "hello" > test.txt
Get-FileHash test.txt -Algorithm SHA256
```

You’ll get a long fingerprint (the hash) for the file. Now change just one character and check again:

```powershell
echo "hello!" > test.txt
Get-FileHash test.txt -Algorithm SHA256
```

I added only a single “!”, but the new fingerprint looked completely different from the first one. That is why SHA-256 is used to check downloads and identify files.

---

### 3. HMAC: proving who sent it (shared secret)

SHA-256 tells you the data didn’t change, but not who made it. An HMAC adds a secret key shared between two people or systems.

The sender makes a tag from the data and the secret. The receiver recomputes the tag with the same secret. If it matches, the data is unchanged and came from someone who knows the secret.

This is common when one service sends messages to another, like a payment provider notifying your app. The catch: both sides hold the same key, so both could create valid tags.

---

### 4. Digital signature: proving who sent it (for everyone)

A digital signature uses two keys. A private key signs the data, and only the owner has it. A public key checks the signature, and anyone can have it.

So anyone can verify, but only the owner can sign. That is how software updates are trusted: the company signs the update, and millions of devices can verify it without being able to forge it.

*Refer to the image here: ./diagrams/03-signatures.svg*
*Refer to the image here: ./diagrams/04-more-types.svg*

---

## Where you already use checksums

You rely on them every day without noticing:
* **Internet traffic:** network frames and TCP/UDP packets carry checksums, so damaged data can be dropped and resent.
* **Downloads:** many software sites publish a SHA-256 value so you can check your file matches.
* **Cloud storage:** when you upload a file, the service can compare checksums to confirm it arrived intact.
* **Databases and disks:** they use checksums to spot corrupted data before it spreads.
* **Backups:** a good backup system checks that what it saved is what it meant to save.

**A common mistake: trusting the wrong checksum.**
Say you download software from a random website, and the same website shows the checksum. You check it and it matches. Great?
Not really. If an attacker replaced the file, they could replace the checksum too. A checksum is only useful when the expected value comes from a source you trust, like the official site or a signed release.

---

## How to choose

Before picking a tool, ask one question:
**What failure am I trying to catch?**

*Refer to the image here: ./diagrams/05-decision-tree.svg*

---

## Quick reference

| Situation                                  | Use               |
| ------------------------------------------ | ----------------- |
| Catching accidents (noise, flipped bits)   | CRC               |
| A strong fingerprint for files/downloads   | SHA-256           |
| Proving who sent it (shared secret)        | HMAC              |
| Proving who sent it (for anyone to verify) | Digital signature |

**Key Takeaways:**
* A checksum is a small value used to detect whether data changed.
* It detects problems but doesn’t fix them. You need a retry, a backup, or another copy.
* A plain checksum protects against accidents, not attackers.
* Never ignore a mismatch. It means the data is not what you expected. Data doesn’t announce when it’s broken. Checksums are how systems find out.
