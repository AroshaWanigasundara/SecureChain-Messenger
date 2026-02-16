# SecureChain Messenger

A decentralized, end-to-end encrypted messaging application built on Substrate blockchain technology. This project demonstrates privacy-preserving communication where message content stays off-chain while cryptographic hashes are stored on-chain for verification.

## 🔒 Overview

SecureChain Messenger combines blockchain technology with modern encryption to provide:

- **End-to-End Encryption**: Messages encrypted using NaCl box (Curve25519, XSalsa20, Poly1305)
- **Blockchain Verification**: Message hashes stored on-chain for tamper-proof verification
- **Privacy-First**: Only cryptographic hashes on-chain, content transmitted peer-to-peer
- **Spam Prevention**: Economic bonds deter malicious actors
- **Contact Management**: Bidirectional approval system for trusted communications

## 📁 Project Structure

```
SecureChain-Messenger/
├── BlockChain-Messaging/     # Substrate blockchain node and runtime
│   ├── node/                 # Node implementation
│   ├── runtime/              # Runtime configuration
│   ├── pallets/
│   │   └── messaging/        # Custom messaging pallet
│   └── README.md             # Blockchain documentation
│
└── Frontend/                 # React-based web application
    ├── src/
    │   ├── components/       # UI components
    │   ├── contexts/         # React contexts (Blockchain, Encryption, PubNub)
    │   ├── lib/              # Utilities and constants
    │   └── pages/            # Application pages
    └── README.md             # Frontend documentation
```

## 🚀 Quick Start

### Prerequisites

- **Blockchain Node**:
  - Rust toolchain (stable)
  - Substrate dependencies
  - Node.js ≥18 (for benchmarking tools)

- **Frontend**:
  - Node.js ≥18
  - npm or yarn
  - Polkadot.js browser extension

### 1. Start the Blockchain Node

```bash
cd BlockChain-Messaging

# Build the node
cargo build --release

# Run in development mode
./target/release/secure-messaging-node --dev --rpc-external --rpc-port 9946
```

The blockchain will be available at `ws://localhost:9946`

### 2. Start the Frontend

```bash
cd Frontend

# Install dependencies
npm install

# Start development server
npm run dev
```

Access the application at `http://localhost:5173`

## 🏗️ Architecture

### Blockchain Layer (Substrate)

**Custom Messaging Pallet** (`pallets/messaging/`)
- **User Profiles**: On-chain public key registry
- **Message Hashes**: Stores Blake2-256 hashes of encrypted messages
- **Contact Management**: Bidirectional approval system
- **Spam Bonds**: 10 UNIT economic deposit for anti-spam
- **Hash Expiry**: 7-day verification window (prevents blockchain bloat)

**Storage**:
- `UserProfiles`: AccountId → PublicKey (max 256 bytes)
- `MessageHashes`: MessageId → (Hash, BlockNumber, Sender, Recipient)
- `ApprovedContacts`: (AccountId, AccountId) → bool
- `SpamBonds`: AccountId → Balance

**Extrinsics**:
- `register_profile(public_key)` - Register encryption key
- `update_profile(public_key)` - Update encryption key
- `send_message_hash(recipient, hash)` - Store message hash
- `approve_contact(contact)` - Add trusted contact
- `remove_contact(contact)` - Remove contact
- `challenge_spam(message_id)` - Report spam
- `refund_bond()` - Reclaim spam bond

### Frontend Layer (React + TypeScript)

**Technology Stack**:
- React 18 with TypeScript
- Polkadot.js API for blockchain interaction
- TweetNaCl for encryption (libsodium-wrappers compatible)
- PubNub for peer-to-peer message delivery
- Tailwind CSS + shadcn/ui for styling
- Vite for development

**Key Components**:
- **BlockchainContext**: Manages wallet, profiles, contacts, messages
- **EncryptionContext**: Handles NaCl key generation and message encryption
- **PubnubContext**: Real-time peer-to-peer message delivery
- **ChatInterface**: Main messaging UI with verification
- **ProfileCard**: Key management and registration
- **ContactList**: Blockchain-synced contact management

## 🔐 Security Model

### Message Flow

1. **Sender** retrieves recipient's public key from blockchain
2. **Sender** encrypts message locally using NaCl box
3. **Sender** computes Blake2-256 hash of encrypted message
4. **Sender** submits hash to blockchain (pays transaction fee)
5. **Sender** transmits encrypted message via PubNub (P2P)
6. **Recipient** receives encrypted message
7. **Recipient** fetches hash from blockchain
8. **Recipient** verifies hash matches (tamper detection)
9. **Recipient** decrypts message using private key

### Threat Mitigation

| Threat | Mitigation |
|--------|-----------|
| Spam attacks | 10 UNIT economic bonds |
| Message tampering | Blake2-256 hash verification |
| Replay attacks | Unique message IDs + timestamps |
| Storage bloat | 7-day hash expiry |
| Unsolicited messages | Bidirectional contact approval |
| Key compromise | Support for key rotation via `update_profile` |

## 🧪 Testing

### Blockchain Tests

```bash
cd BlockChain-Messaging

# Run pallet tests
cargo test -p pallet-messaging

# Run with output
cargo test -p pallet-messaging -- --nocapture
```

**Test Coverage**:
- Profile registration/update
- Message hash sending
- Contact approval/removal
- Spam challenges
- Bond refunds
- Hash verification
- Message expiry

### Frontend Testing

```bash
cd Frontend

# Run tests (if configured)
npm test
```

## 📊 Performance & Benchmarking

### Generate Weights

```bash
cd BlockChain-Messaging

# Build with benchmarks
cargo build --release --features runtime-benchmarks

# Run benchmarks
./target/release/secure-messaging-node benchmark pallet \
    --chain dev \
    --pallet pallet_messaging \
    --extrinsic "*" \
    --steps 50 \
    --repeat 20 \
    --output pallets/messaging/src/weights.rs
```

### Expected Weights
- `register_profile`: ~50M weight units
- `send_message_hash`: ~40M weight units
- `approve_contact`: ~35M weight units

## 🌐 Production Deployment

### Blockchain Node

1. **Build optimized release**:
   ```bash
   cargo build --release
   ```

2. **Configure chain spec**:
   ```bash
   ./target/release/secure-messaging-node build-spec \
       --chain local > customSpec.json
   ```

3. **Deploy with systemd** or Docker

### Frontend

1. **Update RPC endpoint**:
   ```typescript
   // src/lib/constants.ts
   export const RPC_ENDPOINT = "wss://your-node.example.com:9946";
   ```

2. **Build production bundle**:
   ```bash
   npm run build
   ```

3. **Deploy `dist/` folder** to static hosting (Vercel, Netlify, AWS S3)

### PubNub Configuration

Replace placeholder keys in frontend:
```typescript
// src/utils/pubnub.ts
const pubnub = new PubNub({
  publishKey: 'your-publish-key',
  subscribeKey: 'your-subscribe-key',
  uuid: address
});
```

## 🔧 Configuration

### Blockchain Parameters

Edit `runtime/src/configs/mod.rs`:

```rust
parameter_types! {
    // Spam bond amount (10 UNIT = 10,000,000,000,000)
    pub const SpamBond: Balance = 10 * UNIT;
    
    // Max contacts per user
    pub const MaxContactsPerUser: u32 = 1000;
    
    // Message hash expiry (7 days in blocks)
    pub const MessageHashExpiry: BlockNumber = 7 * DAYS;
}
```

### Frontend Constants

Edit `src/lib/constants.ts`:

```typescript
export const RPC_ENDPOINT = "ws://localhost:9946";
export const SPAM_BOND = BigInt("10000000000000");
export const MESSAGE_HASH_EXPIRY_DAYS = 7;
```

## 📚 Documentation

- **Blockchain**: See `BlockChain-Messaging/README.md` for detailed pallet documentation
- **Frontend**: See `Frontend/README.md` for development guide
- **API Reference**: Run `cargo doc --open` in `BlockChain-Messaging/`

## 🐛 Known Limitations

1. **Message Storage**: Messages stored in browser localStorage (MVP)
   - **Production**: Migrate to IPFS or backend server
   
2. **PubNub Free Tier**: Limited message history
   - **Production**: Upgrade to paid plan or implement fallback

3. **Key Management**: No recovery mechanism
   - **Future**: Implement social recovery or hardware wallet integration

4. **Scalability**: Single-chain design
   - **Future**: Implement parachain for Polkadot/Kusama

## 🤝 Contributing

1. Fork the repository
2. Create feature branch (`git checkout -b feature/amazing-feature`)
3. Commit changes (`git commit -m 'Add amazing feature'`)
4. Push to branch (`git push origin feature/amazing-feature`)
5. Open Pull Request

## 📄 License

This project is licensed under the **Unlicense** - see LICENSE files for details.

## 👨‍💻 Author

**Arosha Wanigasundara**

- GitHub: [@arowanas](https://github.com/yourusername)
- Email: [arosha@example.com](mailto:arosha@example.com)

## 🙏 Acknowledgments

- Built with [Substrate](https://substrate.io/) framework
- Encryption by [TweetNaCl](https://tweetnacl.js.org/)
- UI components from [shadcn/ui](https://ui.shadcn.com/)
- Real-time messaging via [PubNub](https://www.pubnub.com/)

## 🔗 Links

- **Live Demo**: [https://securechain-messenger.example.com](https://securechain-messenger.example.com)
- **Documentation**: [https://docs.securechain-messenger.example.com](https://docs.securechain-messenger.example.com)
- **Block Explorer**: [https://explorer.securechain-messenger.example.com](https://explorer.securechain-messenger.example.com)

---

**⚠️ MVP Notice**: This is a proof-of-concept implementation. For production use, implement proper message storage (IPFS/backend), key recovery mechanisms, and comprehensive security audits.
