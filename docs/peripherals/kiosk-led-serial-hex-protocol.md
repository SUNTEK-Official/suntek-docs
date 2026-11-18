# Kiosk LED & Fill-Light Serial HEX Protocol

Applicable to SUNTEK Kiosk terminals with serial LED controller boards connected on `/dev/ttyS5` at **9600 bps (8N1)**.

## Frame Format

- Baud rate: `9600` bps (`8 data bits, No parity, 1 stop bit`)
- Fixed header/tail control frame (`4C <CMD> 23`):
  - `4C 64 23` — Turn fill light **ON**
  - `4C 65 23` — Turn fill light **OFF**
  - RGB status commands use 3-byte RGB payload frames aligned with `LED Communication Command V6.5`.
