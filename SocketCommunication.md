# Socket communucation protocol

## Overview
The server only keeps the latest communication channel info between it and each client. The client can query the latest channel from the server and continue on from there. Upon transferring all the balance in the channel, the client sends a finalized transmission and starts a new channel when the user buys more hours.

After receiving each payment in the channel, the server sends the secret for downloading the next n seconds of the stream.

## The algorithm

### Idle state
1. User clicks play
2. Client queries the server for server's public key
3. Client queries the server for the latest active channel info
4. If a channel does not exist,
4.1. Client promopts the user to buy a subcription (more stream time)
4.2. Client creates a payment channel, keeps the info and sends the info to the server
5. If a channel already exists, client verifies the info and keeps it
6. Client sends a "play" message to the server and enters streaming state

### Streaming state
1. Client receives a "pay" message from the server
2. Client performs the payment and signing in the channel and sends the info to the server. If the resulting balance is zero, the state is signed as a final state.
3. The server sends the decryption key for the next n seconds of the stream
4. If the balance is zero, the client forgets the channel and enters the idle state

## Messages and responces

**publicKey**
- Client only sends
- Server replies on the same event with the server's public key and wallet address

**latestState**
- Client sends it along with its publicKey
- Server replies with the latest state object or empty: channel_id, channel_address, client_wallet_address, client_public_key, client_balance, server_balance, client_sequence_number, client_signature

**newChannel**
- Client sends it to server with the latest state (which is actually the first state)
- Server replies with **latestState**

**play**
- Client sends it to server along with client's publicKey
- Server does not reply but enters in streaming state with that client

**pay**
- Server only sends
- Client replies with a signature of the updated state which is the next payment
- If server does not receive the response to pay, it will enter idle state with that client. The client will have to send **play** again

**secret**
- Server sends it to client along with the secret to download the next n seconds of the stream
- Client does not reply but uses the secret to continue with the stream
