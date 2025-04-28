<!--
@eval
<script>
import('https://esm.sh/nostr-tools@1.17.0').then(async (nostr) => {
@input
send.lia("LIA: stop")
})

"LIA: wait"
</script>
@end
-->

# Nostr

``` javascript
// 1. Generate a new private key (hex-encoded string)
const privKey = nostr.generatePrivateKey()
// 2. Derive the corresponding public key
const pubKey = nostr.getPublicKey(privKey)

console.log("Private Key: ", privKey)
console.log("Public Key: ", pubKey)

console.debug("📋 Store your private key somewhere safe!")
```
@eval

---------------

``` javascript
// A more robust Nostr posting example

const privateKey = nostr.generatePrivateKey()
const publicKey = nostr.getPublicKey(privateKey)

console.log('Using public key:', publicKey)

try {
  if (nostr.SimplePool) {
    console.log('Using SimplePool approach')
    const pool = new nostr.SimplePool()
    
    // Add more relays for better reliability
    const relays = [
      'wss://relay.damus.io',
      'wss://nos.lol',
      'wss://relay.nostr.info',
      'wss://nostr.fmt.wiz.biz',
      'wss://relay.nostr.band'
    ]
    
    // Create a note event
    const event = {
      kind: 1,
      created_at: Math.floor(Date.now() / 1000),
      tags: [],
      content: 'Hello from Nostr using SimplePool! ' + new Date().toISOString(),
      pubkey: publicKey,
    }
    
    // Sign the event
    event.id = nostr.getEventHash(event)
    event.sig = nostr.signEvent(event, privateKey)
    
    console.log('Event created:', event)
    console.log('Publishing to relays:', relays)
    
    // Properly handle the Promise returned by publish
    try {
      // Publish returns an array of promises
      const pubPromises = pool.publish(relays, event)
      console.log('Publication requested to', relays.length, 'relays')
      
      // Track which relays succeeded
      const results = { success: [], failed: [] }
      
      // Handle the promises more explicitly
      Promise.allSettled(pubPromises).then(outcomes => {
        outcomes.forEach((outcome, i) => {
          if (outcome.status === 'fulfilled') {
            console.log(`✅ Published to ${relays[i]}`)
            results.success.push(relays[i])
          } else {
            console.log(`❌ Failed to publish to ${relays[i]}: ${outcome.reason}`)
            results.failed.push(relays[i])
          }
        })
        
        if (results.success.length > 0) {
          console.log(`Successfully published to ${results.success.length} relays`)
        } else {
          console.error('Failed to publish to any relays')
        }
      })
    } catch (pubError) {
      console.error('Publish error:', pubError)
      //out.textContent += '\n\nFailed to send message: ' + (pubError.message || pubError)
    }
  } else {
    console.error('SimplePool not available in this version of nostr-tools')
    //out.textContent += '\n\nFailed: SimplePool not available in this version'
  }
} catch (error) {
  console.error('Error:', error)
  //out.textContent += '\n\nError: ' + (error.message || error)
}
```
<script>
import('https://esm.sh/nostr-tools@1.17.0').then(async (nostr) => {
@input
//send.lia("LIA: stop")
})

"LIA: wait"
</script>

-----------------

``` javascript
// Subscribing only to your own content

// Initialize connection
const privateKey = nostr.generatePrivateKey()
const publicKey = nostr.getPublicKey(privateKey)

console.log('Using public key for identification:', publicKey)

try {
  if (nostr.SimplePool) {
    console.log('Creating SimplePool for subscriptions...')
    const pool = new nostr.SimplePool()
    
    // Use multiple relays for better content discovery
    const relays = [
      'wss://relay.damus.io',
      'wss://nos.lol',
      'wss://relay.nostr.info',
      'wss://nostr.fmt.wiz.biz'
    ]
    
    console.log('Subscribing to only MY events from relays')
    
    // This filter will only show your own posts
    const filters = [
      {
        kinds: [1],
        authors: [publicKey], // Only your public key
        since: Math.floor(Date.now() / 1000) - 3600, // Last hour
        limit: 10
      }
    ]
    
    console.log('Creating subscription with filters:', filters)
    const sub = pool.sub(relays, filters)
    
    // Handle events as they come in
    sub.on('event', event => {
      console.log(`📬 Received my note:`)
      console.log(`  Content: ${event.content.slice(0, 100)}${event.content.length > 100 ? '...' : ''}`)
      console.log(`  Posted: ${new Date(event.created_at * 1000).toLocaleString()}`)
      console.log('-'.repeat(40))
    })
    
    // Handle when a relay is done sending events
    sub.on('eose', () => {
      console.log('End of stored events - now listening for new events in real-time')
    })
    
    // Publish a test note
    setTimeout(async () => {
      console.log('Publishing a test note...')
      
      const event = {
        kind: 1,
        created_at: Math.floor(Date.now() / 1000),
        tags: [],
        content: 'This is a test from my filtered subscription! ' + new Date().toISOString(),
        pubkey: publicKey,
      }
      
      event.id = nostr.getEventHash(event)
      event.sig = nostr.signEvent(event, privateKey)
      
      await pool.publish(relays, event)
      console.log('Published test note! It should appear in the subscription feed shortly.')
    }, 3000)
    
    // Cleanup after 30 seconds
    setTimeout(() => {
      console.log('Closing subscription...')
      sub.unsub()
      console.log('Subscription closed')
    }, 30000)
  }
} catch (error) {
  console.error('Error:', error)
}
```
<script>
import('https://esm.sh/nostr-tools@1.17.0').then(async (nostr) => {
@input
//send.lia("LIA: stop")
})
"LIA: wait"
</script>