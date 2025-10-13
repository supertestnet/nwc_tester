# NIP-44 Support Implementation

## Overview

This NWC Tester now supports both NIP-04 and NIP-44 encryption standards as specified in NIP-47.

## Changes Made

### 1. Added nostr-tools Library

- Imported `nostr-tools` v2.7.2 which provides industry-standard NIP-44 encryption support
- Library is loaded as an ES module and made available globally as `window.nostrTools`

### 2. Encryption Detection

- `detectWalletEncryption(nwc_info)` - Automatically detects which encryption schemes the wallet service supports by:
  - Fetching the wallet's info event (kind 13194)
  - Parsing the `encryption` tag
  - Caching results to avoid repeated queries
  - Defaulting to NIP-04 for legacy wallet services without encryption tags

### 3. Encryption/Decryption Functions

- `encryptNIP44(privkey, pubkey, text)` - Encrypts using NIP-44 (ChaCha20-Poly1305)
- `decryptNIP44(privkey, pubkey, ciphertext)` - Decrypts NIP-44 encrypted content
- `encryptMessage(nwc_info, text, encryption_type)` - Wrapper that chooses NIP-04 or NIP-44
- `decryptMessage(privkey, pubkey, ciphertext, encryption_type)` - Wrapper that chooses appropriate decryption

### 4. Enhanced NWC Request Functions

All NWC commands now use enhanced versions that:

- Automatically detect wallet encryption support
- Prefer NIP-44 when available, fallback to NIP-04
- Include `["encryption", "nip44_v2"]` tag in requests when using NIP-44
- Handle responses with the correct decryption method

Enhanced functions:

- `sendNWCRequest()` - Generic request sender with encryption negotiation
- `getEnhancedResponse()` - Response handler with proper decryption
- `enhancedGetInfo()`
- `enhancedGetBalance()`
- `enhancedMakeInvoice()`
- `enhancedCheckInvoice()`
- `enhancedPayInvoice()`
- `enhancedListTransactions()`

### 5. Notification Support

- `listenForNotification()` - Listens for both kind 23196 (NIP-04) and 23197 (NIP-44) notifications
- Automatically decrypts notifications based on event kind
- Displays which encryption method was used in the UI

### 6. UI Updates

- Get Info section now displays:
  - NIP-44 support status (✅/❌)
  - NIP-04 support status (✅/❌)
  - Preferred encryption method
- All notification displays show which encryption was used (NIP-44 or NIP-04)

## Encryption Negotiation Flow

1. **Initial Connection**: When a user provides an NWC connection string
2. **Detection**: Tester fetches wallet's info event (kind 13194) and checks `encryption` tag
3. **Preference**: If wallet supports both, NIP-44 is preferred
4. **Request**: All requests include appropriate encryption tag based on negotiation
5. **Response**: Responses are decrypted using the method specified in the request
6. **Notifications**: Tester listens for both notification kinds and decrypts appropriately

## Backwards Compatibility

The tester maintains full backwards compatibility:

- Works with legacy NIP-04-only wallet services
- Automatically falls back to NIP-04 if NIP-44 is not supported
- Handles wallet services without `encryption` tags (assumes NIP-04)

## Testing

To test NIP-44 support:

1. Use an NWC connection string from a wallet service that supports NIP-44
2. Click "Get wallet info" - should show "NIP-44: ✅ Supported" and "Preferred: NIP44_V2"
3. All subsequent operations will use NIP-44 encryption
4. Check console logs to see encryption method being used
5. Notification displays will show "(NIP-44)" or "(NIP-04)" to indicate which was used

## Security Benefits of NIP-44

- **Modern Cryptography**: Uses ChaCha20-Poly1305 instead of AES-CBC
- **Authenticated Encryption**: Prevents tampering and forgery
- **Better Security Properties**: Addresses known weaknesses in NIP-04
- **Constant-Time Operations**: Better resistance to timing attacks

## Console Logging

The implementation includes debug logging:

- "Wallet supports: [encryption_types]"
- "Listening for notification kinds: [kinds]"
- "Sending {method} request with {encryption_type} encryption"
- "Decrypting NIP-44/NIP-04 notification"
