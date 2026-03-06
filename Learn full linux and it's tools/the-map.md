# The Map
> A personal reference of tools, philosophies, communities, and rabbit holes.  
> Nothing here is explained fully. Everything here is worth searching.

---

## Who You Are (The Bigger Picture)

- **Hacker** — original meaning. Someone who understands systems deeply, finds elegant solutions, refuses artificial limitations. Not a criminal. A craftsperson.
- **Cypherpunk** — hacker + privacy ideology. Believes cryptography and code are tools of liberation. Tor, PGP, Signal, Bitcoin all came from this movement.
- **Free Software Advocate** — Richard Stallman's ethical framework. Software freedom is a human rights issue. Four freedoms: run, study, modify, redistribute.
- **Digital Sovereigntist** — you own your data, your infrastructure, your identity. No corporation is a middleman.
- **Unix Philosopher** — do one thing well, compose tools, understand what you run. A way of thinking, not just coding.
- **Homelab Enthusiast** — runs real infrastructure at home for learning, privacy, and utility.

---

## The Worldview / Ideology Layer

- **The Hacker Ethic** — Steven Levy, 1984. Information wants to be free. Distrust authority. Promote decentralization.
- **Cypherpunk Manifesto** — Eric Hughes, 1993. Privacy through cryptography. Don't ask permission.
- **GNU Manifesto** — Stallman, 1985. Why free software matters ethically.
- **Cathedral and the Bazaar** — Eric Raymond. Open source development philosophy.
- **Zen and the Art of Motorcycle Maintenance** — Robert Pirsig. Quality, craft, understanding systems. Not tech but deeply relevant.
- **Whole Earth Catalog** — Stewart Brand. The original "tools for living" movement. Everything that followed descends from this.
- **Effective Altruism** — unrelated ideology but worth knowing. Use your skills to do maximum good.
- **Stoicism** — ancient philosophy. Control only what you can control. Relevant to OPSEC, minimalism, focus.
- **Minimalism** — own less, own better. Applies to software, hardware, possessions.

---

## Operating Systems

### Linux
- **Arch Linux** — DIY, rolling release, you own every decision
- **NixOS** — entire system declared in one config file. Reproducible, rollbackable
- **Gentoo** — compile everything from source. Maximum control, maximum time
- **Void Linux** — independent, no systemd, runit init
- **Alpine Linux** — tiny, musl-based, used in containers
- **Immutable distros** — Fedora Silverblue, VanillaOS, Bluefin. Read-only OS, apps in containers

### Security-Focused OS
- **Qubes OS** — compartmentalization via VMs. Each app is isolated. Snowden's OS.
- **Tails** — amnesic, runs from USB, leaves zero trace, routes through Tor
- **Whonix** — two-VM setup: Tor gateway + workstation. IP-safe even if workstation is compromised
- **Kali / BlackArch / ParrotOS** — penetration testing distributions

### Mobile OS
- **GrapheneOS** — hardened Android, Pixel only. The gold standard
- **CalyxOS** — easier than Graphene, MicroG built in
- **DivestOS** — more device support, privacy-hardened LineageOS fork
- **LineageOS** — AOSP fork, widest device support
- **postmarketOS** — actual Linux on phones. Alpine-based
- **/e/OS** — degoogled, user-friendly
- **Ubuntu Touch** — community-maintained mobile Ubuntu

### Experimental / Niche
- **Plan 9** — Bell Labs OS, everything is a file, influenced Unix deeply
- **Haiku** — BeOS successor, fascinating design
- **Redox OS** — written in Rust, microkernel
- **SerenityOS** — built from scratch by one person (Andreas Kling), now a community

---

## Hardware

### Laptops / Computers
- **Framework** — modular, repairable, Linux-first
- **System76** — Linux-native hardware, open firmware (coreboot)
- **Purism Librem** — hardware kill switches, open firmware, extreme end
- **ThinkPads** — legendary Linux compat, cheap second-hand, fully repairable
- **PINE64 PineBook Pro** — ARM laptop, cheap, open hardware

### Phones
- **Google Pixel** — only for GrapheneOS. Ironic but the hardware is open enough
- **PinePhone / PinePhone Pro** — actual Linux phone, rough but real
- **Fairphone** — most repairable mainstream phone, ethical supply chain

### Single Board Computers
- **Raspberry Pi** — the gateway drug
- **Orange Pi 5** — faster, cheaper than RPi
- **Radxa Rock** series
- **BeagleBone** — hacker-oriented, more GPIO than RPi
- **ODROID** series

### Security Hardware
- **YubiKey** — hardware 2FA, phishing-proof, FIDO2/GPG/SSH
- **NitroKey** — open source YubiKey alternative
- **Librem Key** — works with Purism hardware for boot verification
- **USB data blocker** — charge from public ports without data risk
- **Faraday bag** — blocks all RF, no tracking possible

### RF / SDR Hardware
- **RTL-SDR** — $30 dongle, receive radio: aircraft, ships, weather satellites, pagers
- **HackRF One** — transmit and receive, full SDR research tool
- **Flipper Zero** — RFID, NFC, IR, sub-GHz, GPIO multi-tool
- **Proxmark3** — deep RFID/NFC research and cloning
- **YARD Stick One** — sub-GHz radio tool
- **WiFi Pineapple** — wireless auditing platform
- **LAN Turtle** — network implant for pentesting

### Networking Hardware
- **Old PC as router** — run OPNsense/pfSense on it, replace ISP garbage
- **Managed switches** — VLANs, port mirroring, traffic inspection
- **POE switches** — power your APs and cameras over ethernet
- **NAS builds** — TrueNAS on custom hardware with ZFS

---

## Networking

### Concepts
- **OSI model** — 7 layers of how networks work. Know all of them.
- **TCP/IP, UDP** — the actual protocols everything runs on
- **Subnetting / CIDR** — how IP addresses are divided
- **VLANs** — segment your network logically. IoT away from personal machines
- **DNS** — how names become IPs. Understand it deeply, not just Pi-hole
- **BGP** — how the internet actually routes traffic between networks
- **DNSSEC** — cryptographic verification of DNS records
- **Zero trust networking** — trust nothing inside or outside by default

### Tools
- **OPNsense / pfSense** — real firewall OS. Replace your ISP router
- **Unbound** — self-hosted recursive DNS. Your queries go nowhere
- **Suricata / Snort** — intrusion detection system
- **Zeek** — network traffic analyzer
- **Wireshark / tcpdump** — read actual packets
- **nmap** — port scanning, network discovery
- **Tailscale** — WireGuard-based mesh VPN (you have this)
- **Nebula** — Slack's open source overlay network, self-hosted Tailscale
- **WireGuard** — the protocol under Tailscale. Learn it raw
- **Tor** — onion routing, anonymity network
- **I2P** — alternative anonymous network, different threat model
- **Yggdrasil** — end-to-end encrypted mesh network, experimental internet

---

## Security

### Concepts
- **Threat modeling** — who is your adversary, what do they want, what can they do? Do this before any security decision
- **OPSEC** — operational security. Not just tools, behavior
- **Principle of least privilege** — minimum permissions for everything
- **Defense in depth** — layers, never single point of security
- **Supply chain attacks** — the real modern threat. Trust the source, not just the software
- **Air gap** — some things should never touch a network
- **Zero knowledge** — cryptographic proof without revealing the secret itself
- **Forward secrecy** — compromise of long-term key doesn't compromise past sessions
- **Reproducible builds** — can your binary be verified against source code?

### Cryptography
- **GPG/PGP** — encrypt files, sign commits, verify software
- **AES, ChaCha20** — symmetric encryption
- **RSA, ECC, Ed25519** — asymmetric encryption / key pairs
- **TLS/SSL** — how HTTPS actually works
- **LUKS** — full disk encryption on Linux. Should be running right now
- **Age** — modern, simple encryption tool. Better than GPG for files
- **Veracrypt** — encrypted containers, hidden volumes, plausible deniability

### Practice / Learning
- **OverTheWire** — wargames starting with Bandit (you're here)
- **TryHackMe** — guided learning paths, beginner friendly
- **HackTheBox** — harder, more realistic machines
- **PicoCTF** — beginner CTF challenges
- **CTF competitions** — Capture The Flag. Real hacking competitions
- **OWASP Top 10** — ten most common web vulnerabilities every dev must know
- **CVE database** — real vulnerability disclosures. Reading these teaches a lot

---

## Privacy

### Concepts
- **Data minimization** — generate and store as little data as possible
- **Compartmentalization** — separate identities for separate activities
- **Metadata** — often more revealing than content. Strip it
- **Browser fingerprinting** — you're trackable even without cookies
- **Traffic analysis** — even encrypted traffic reveals patterns

### Tools
- **LibreWolf / Firefox hardened** — private browsers
- **uBlock Origin** — the only ad/tracker blocker that matters
- **Mullvad / ProtonVPN** — trustworthy VPNs for public networks
- **Signal** — encrypted messaging
- **Matrix / Element** — self-hosted encrypted chat
- **Proton ecosystem** — Mail, Drive, Calendar, VPN, Pass
- **SimpleX** — no user IDs at all, newer messaging protocol
- **Session** — no phone number required, decentralized
- **Briar** — peer-to-peer messaging, works without internet over Bluetooth/WiFi
- **Monero** — private cryptocurrency. Bitcoin is not private
- **mat2 / exiftool** — strip metadata from files before sharing

---

## Self-Hosting

### Infrastructure
- **Proxmox** — VM and container hypervisor. The homelab standard
- **TrueNAS** — NAS OS with ZFS. Snapshots, replication, data integrity
- **Home Assistant** — local home automation, no cloud
- **Docker / Podman** — containerize everything
- **Kubernetes / K3s** — container orchestration at scale. K3s for homelab
- **Portainer** — Docker GUI management
- **Cloudflare Tunnel** — expose services without opening ports
- **Wireguard** — self-hosted VPN alternative to Tailscale

### Services to Self-Host
- **Nextcloud** — Google Drive/Docs replacement
- **Immich** — Google Photos replacement
- **Jellyfin** — Netflix replacement (you have)
- **Navidrome** — Spotify replacement
- **Vaultwarden** — Bitwarden server, password manager (you have)
- **SearXNG** — Google Search replacement (you have)
- **Invidious** — YouTube frontend proxy
- **Redlib** — Reddit frontend proxy
- **Miniflux / FreshRSS** — RSS reader. Follow anything without algorithms
- **Calibre-Web** — ebook library
- **Paperless-ngx** — document management, OCR, full text search
- **Stirling-PDF** — every PDF tool, self-hosted
- **Mealie** — recipe manager
- **Gitea / Forgejo** — self-hosted GitHub
- **Woodpecker CI** — self-hosted CI/CD
- **Gotify / Ntfy** — self-hosted push notifications
- **Umami / Plausible** — privacy-respecting analytics
- **Mailcow / Maddy** — self-hosted email (advanced, painful but possible)
- **Uptime Kuma** — self-hosted status page and monitoring
- **Prometheus + Grafana + Loki** — monitoring, dashboards, log aggregation

---

## Linux & Terminal

### Shell
- **Bash scripting** — real scripts, not just one-liners
- **zsh** + plugins (autosuggestions, syntax-highlighting)
- **fish** — friendlier shell, controversial
- **starship** — cross-shell prompt
- **atuin** — shell history as a searchable database, syncs across machines

### Navigation
- **zoxide** — smarter cd, learns your habits
- **fzf** — fuzzy finder, plugs into everything
- **yazi / lf / ranger** — terminal file managers
- **broot** — interactive tree search

### Viewing
- **bat** — better cat
- **eza / lsd** — better ls
- **delta** — beautiful git diffs
- **glow** — render markdown in terminal

### Searching
- **ripgrep (rg)** — fastest grep, respects .gitignore
- **fd** — better find
- **sd** — better sed

### System Monitoring
- **btop** — beautiful process monitor
- **dust** — better du
- **duf** — better df
- **bandwhich** — bandwidth per process
- **procs** — better ps

### Git
- **lazygit** — TUI for git, essential
- **tig** — text-mode git log
- **gh** — GitHub CLI
- **delta** — diff viewer

### Productivity
- **tmux** — terminal multiplexer (you have)
- **zellij** — newer tmux alternative, friendlier config
- **taskwarrior** — CLI task manager
- **pass / gopass** — git-backed CLI password manager

### Text Processing
- **jq** — JSON in terminal
- **yq** — YAML in terminal
- **awk, sed** — classic text processing, learn properly
- **miller (mlr)** — CSV/JSON/YAML data processing

---

## Programming & Engineering

### Languages Worth Knowing
- **C** — understand how computers actually work. Everything else sits on this
- **Rust** — C but memory safe. The future of systems programming
- **Go** — simple, concurrent, compiles to single binary. Used by Docker, Kubernetes
- **Python** — glue language, automation, data, AI/ML ecosystem
- **Lua** — tiny, embeddable. Neovim's scripting language. NeoVim plugins, game modding
- **JavaScript / TypeScript** — unavoidable for web. TypeScript makes it bearable
- **Shell (Bash)** — for automation, not applications
- **SQL** — not optional. Every serious application uses a database
- **Assembly** — for understanding. x86 and ARM basics change how you think
- **Zig** — newer C alternative, manual memory, very interesting

### Concepts
- **Memory management** — stack vs heap, manual vs GC vs borrow checker
- **Concurrency vs parallelism** — different problems, different solutions
- **Systems programming** — writing software that talks directly to hardware/OS
- **Compilers** — how code becomes machine instructions. Build one small one
- **Operating system internals** — syscalls, kernel space, user space
- **Computer architecture** — registers, cache, pipelines, why hardware matters to software

### Resources
- **Nand2Tetris** — build a computer from logic gates. Mind-altering, free
- **OSTEP** — best OS book, free (ostep.org)
- **K&R C** — the book on C
- **The Rust Book** — free at doc.rust-lang.org
- **SICP** — Structure and Interpretation of Computer Programs. MIT classic, changes how you think
- **Build Your Own X** — implement Redis, Git, Docker, SQLite from scratch

---

## Electronics

### Concepts
- **Datasheets** — reading them is the real skill
- **Serial protocols** — UART, SPI, I2C, CAN
- **Interrupts, DMA, timers** — bare metal programming concepts
- **Logic levels** — 3.3V vs 5V, level shifting
- **PCB design** — KiCad (free, powerful)

### Tools
- **Logic analyzer** — see digital signals
- **Oscilloscope** — see analog signals. Even a cheap DSO is transformative
- **Multimeter** — basic but essential
- **Soldering iron** — learn to solder properly
- **KiCad** — schematic capture and PCB layout

### Beyond Arduino
- **Bare metal AVR/STM32** — no Arduino libraries, just you and the registers
- **ESP-IDF** — official Espressif SDK for ESP32, deeper than Arduino
- **Zephyr RTOS** — real-time OS for embedded, used industrially
- **FreeRTOS** — simpler RTOS, widely used
- **Buildroot / Yocto** — custom embedded Linux

### RF / Radio
- **SDR (Software Defined Radio)** — process radio signals in software
- **Amateur radio (HAM)** — licensed, but opens up a whole world of RF
- **LoRa / LoRaWAN** — long range, low power wireless. IoT without WiFi
- **Zigbee / Z-Wave** — local smart home protocols, no cloud
- **MQTT** — lightweight messaging protocol for IoT

---

## Decentralized & Alternative Internet

### Protocols
- **ActivityPub** — protocol powering the Fediverse
- **Nostr** — cryptographic identity, no central servers
- **IPFS** — content-addressed distributed storage
- **Gemini** — simpler web protocol, text-focused, no tracking possible
- **Dat / Hypercore** — peer-to-peer data protocol
- **Tor hidden services (.onion)** — host services with no IP exposure

### Platforms
- **Mastodon** — federated Twitter
- **Pixelfed** — federated Instagram
- **PeerTube** — federated YouTube
- **Lemmy** — federated Reddit
- **Funkwhale** — federated music
- **Misskey / Calckey** — federated social, more features than Mastodon

### Concepts
- **Federation** — servers talk to each other, you own your instance
- **Decentralization** — no single point of control or failure
- **Censorship resistance** — content can't be taken down by one actor
- **Cryptographic identity** — your keys are your identity, not your email

---

## Automation

- **Ansible** — automate server config. Your entire system as code
- **Terraform / OpenTofu** — infrastructure as code. Provision anything
- **n8n** — visual workflow automation (you have)
- **Node-RED** — flow-based automation, great for IoT
- **Home Assistant automations** — physical world automation
- **Systemd timers** — replace cron, more powerful
- **GitHub Actions / Gitea CI** — automate software pipelines
- **Makefile** — simple build automation, underrated

---

## Backup & Data Safety

- **3-2-1 rule** — 3 copies, 2 media types, 1 offsite
- **Restic** — best backup tool. Encrypted, deduplicated, any backend
- **BorgBackup** — alternative, very efficient
- **rclone** — sync to any cloud provider, encrypted
- **ZFS** — filesystem with built-in snapshots and data integrity
- **Btrfs** — Linux-native CoW filesystem with snapshots
- **Backblaze B2** — cheap offsite object storage, rclone compatible

---

## Dotfile Culture & Ricing

- **Dotfiles** — your configuration files. Should be in a git repo
- **chezmoi / stow** — manage dotfiles across machines
- **Nix home-manager** — declare your entire user environment in Nix
- **r/unixporn** — people share setups, you discover tools from screenshots
- **Hyprland** — Wayland compositor (you have)
- **waybar / eww** — custom status bars
- **rofi / wofi / tofi** — app launchers
- **pywal / matugen** — generate colorschemes from wallpapers
- **Neovim** — editor (you have), but depth is infinite
- **Helix** — modal editor like Neovim but different philosophy, built in Rust

---

## Philosophies Beyond Tech

These seem unrelated but they're not.

- **Stoicism** — control only what you control. Resilience. Marcus Aurelius, Epictetus
- **Taoism** — flow, simplicity, doing without forcing. The Unix philosophy is unconsciously Taoist
- **Anarchism** — political philosophy of no rulers. Cypherpunks have strong anarchist roots
- **Solarpunk** — optimistic future where technology and nature coexist. FOSS and sustainability
- **Biopunk** — applying hacker ethics to biology. DIY biotech
- **Luddism (modern)** — not anti-technology, but pro-intentional technology. Use tools that serve you, not the reverse
- **Effective Altruism** — use skills and resources to do maximum measurable good
- **Deep work** — Cal Newport. Sustained, distraction-free focus as a skill worth building
- **Second brain** — Tiago Forte. External system for storing and connecting knowledge (Obsidian fits here)

---

## Communities

- **r/selfhosted** — your people
- **r/homelab** — hardware and infrastructure
- **r/privacy** — privacy tools and practices
- **r/degoogle** — replacing Google services
- **r/unixporn** — setups and ricing
- **r/netsec** — network security
- **r/commandline** — CLI tools
- **Hacker News (news.ycombinator.com)** — tech news, high signal if you filter well
- **Lobsters (lobste.rs)** — like HN but invite-only, higher quality
- **tildeverse** — network of small Unix-based communities, actual public access Unix systems
- **SDF Public Access Unix** — oldest public access Unix system still running
- **2600 Magazine** — hacker magazine, been running since 1984
- **DEF CON / HOPE** — the hacker conferences. Talks are on YouTube

---

## Things That Don't Fit Categories

- **Amateur (HAM) radio** — license required but opens up RF, emergency comms, satellites
- **Mesh networking** — goTenna, Meshtastic (LoRa-based). Communication without internet
- **ReMarkable / Supernote** — e-ink writing tablets for people who hate screens but love notes
- **Mechanical keyboards** — custom firmware (QMK/ZMK), open hardware (you mentioned ergonomic splits)
- **3D printing** — design and print your own hardware enclosures, parts
- **CNC / laser cutting** — fabricate physical things from digital designs
- **Lock picking** — physical security, TOOOL community, "locksport"
- **OSINT** — open source intelligence. Finding information using only public sources
- **Bug bounty** — get paid to find vulnerabilities in real systems
- **CTF (Capture The Flag)** — hacking competitions, best way to learn offensive security
- **Geocaching** — physical treasure hunt with GPS coordinates. Old but active community
- **Wardriving** — mapping WiFi networks while driving. Old tradition, still done
- **Retrocomputing** — restoring and running old hardware. Understanding computing history

---

## What To Search When You Want More

- `awesome-selfhosted` — master list of self-hostable software
- `awesome-cli-apps` — CLI tools list
- `awesome-tuis` — terminal UI applications
- `awesome-privacy` — privacy tools and services
- `awesome-security` — security tools
- `awesome-sysadmin` — sysadmin tools
- `privacy guides (privacyguides.org)` — maintained, practical privacy advice
- `The New Oil (thenewoil.org)` — beginner privacy guide
- `suckless.org` — software that sucks less, extreme minimalism philosophy
- `cat-v.org` — harmful software and complexity criticism

---

*This document is a map, not a manual. Every item here is a door. Open the ones that pull you.*
