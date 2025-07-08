# Iframe Communication Fix Documentation

## Problem Summary
The iframe wallet application was not responding to messages from the parent application. The iframe was only listening for `FlowChange` messages but the parent was sending different message types like `wallet:action:change`, `parent:ready`, and `wallet:state`.

## Root Cause
The iframe's message listener in `src/lib/demo/index.ts` was only configured to handle legacy `FlowChange` messages. It had no handlers for the wallet playground message protocol expected by the parent application.

## Solution Overview
Extended the iframe's message handling system to support the wallet playground message protocol while maintaining backward compatibility with existing `FlowChange` messages.

## Changes Made

### 1. Extended Message Types (`src/lib/demo/DemoConstants.ts`)
- Added `WalletPlaygroundMessage` type for parent-to-iframe communication
- Added `PlaygroundState` type for tracking wallet state
- Extended `DemoFlowType` to include `'idle'` state
- Added `idle` route mapping to handle default state

### 2. Enhanced Message Handling (`src/lib/demo/index.ts`)
- **Added playground state tracking**: Global state to track connection status, address, and selected action
- **Extended message listener**: Now handles both legacy `FlowChange` and new wallet playground messages
- **Added message handlers** for all required message types:
  - `parent:ready` → responds with `playground:ready`
  - `wallet:action:change` → updates selected action and navigates to appropriate route
  - `wallet:state` → updates full playground state
  - `wallet:connect:response` → handles connection responses
  - `wallet:transaction:response` & `wallet:sign:response` → handles transaction responses
  - `wallet:disconnect` → resets state and returns to idle

### 3. Added Testing Utilities (`src/lib/demo/playground-test.ts`)
- Browser console testing functions for debugging
- Simulates parent messages to verify iframe responses
- Provides state inspection and message logging capabilities

## Message Protocol Implementation

### Handshake Flow
```javascript
// Parent sends when iframe loads
{ type: 'parent:ready' }

// Iframe responds with current state
{ 
  type: 'playground:ready',
  data: {
    connected: true,
    address: 'NQ57 2814 7L5B NBBD 0EU7 EL71 HXP8 M7H8 MHKD',
    selectedAction: 'idle'
  }
}
```

### Action Change Flow
```javascript
// Parent sends when user selects action
{ 
  type: 'wallet:action:change',
  data: { action: 'buy' } // or 'stake', 'swap', 'idle'
}

// Iframe navigates to appropriate route and updates state
```

### State Update Flow
```javascript
// Parent sends complete state update
{
  type: 'wallet:state',
  data: {
    connected: false,
    address: null,
    selectedAction: 'idle'
  }
}
```

## Testing the Fix

### 1. Manual Testing in Browser Console

When the iframe is loaded in demo mode, testing utilities are available:

```javascript
// Enable message logging to see all communication
window.playgroundTest.enableMessageLogging();

// Test handshake flow
window.playgroundTest.testHandshakeFlow();

// Test individual action changes
window.playgroundTest.testActionChange('buy');
window.playgroundTest.testActionChange('stake');
window.playgroundTest.testActionChange('swap');
window.playgroundTest.testActionChange('idle');

// Test all actions sequentially
window.playgroundTest.testAllActions();

// Check current state
window.playgroundTest.debugGetState();

// Test sending messages to parent
window.playgroundTest.testSendToParent();
```

### 2. Integration Testing

1. **Load the iframe** in demo mode
2. **Open browser dev tools** and check console for:
   ```
   [Test] Playground test utilities loaded. Use window.playgroundTest in console.
   ```
3. **Send a parent:ready message** and verify response:
   ```javascript
   window.playgroundTest.simulateParentMessage('parent:ready');
   // Should see: [Demo] Parent is ready, sending playground:ready response
   ```
4. **Test action changes** and verify navigation:
   ```javascript
   window.playgroundTest.testActionChange('buy');
   // Should navigate to /buy route
   ```

### 3. Parent-Iframe Integration Test

From the parent application:
1. **Send parent:ready** after iframe loads
2. **Listen for playground:ready** response
3. **Send wallet:action:change** messages
4. **Verify iframe** navigates to correct routes

## Route Mappings

The iframe now supports these action-to-route mappings:

| Action | Route | Description |
|--------|-------|-------------|
| `idle` | `/` | Default wallet state |
| `buy` | `/buy` | Buy NIM flow |
| `stake` | `/staking` | Staking flow |
| `swap` | `/swap/NIM-BTC` | Swap flow |

## Backward Compatibility

The fix maintains full backward compatibility:
- Legacy `FlowChange` messages still work as before
- Existing demo functionality is preserved
- No breaking changes to existing code

## Debug Information

### Console Logging
The iframe now logs all wallet playground messages:
```
[Demo] Received wallet playground message: wallet:action:change { action: 'buy' } from: https://wallet.nimiq.com
[Demo] Changing action to: buy
```

### State Inspection
Current playground state can be inspected via:
```javascript
window.playgroundTest.debugGetState();
```

### Message Logging
Enable comprehensive message logging:
```javascript
window.playgroundTest.enableMessageLogging();
```

## Error Handling

The fix includes comprehensive error handling:
- **Invalid message data**: Warns and skips processing
- **Unknown message types**: Logs warning for unrecognized messages
- **Navigation errors**: Catches and logs routing failures
- **State update errors**: Prevents crashes on malformed state updates

## Security Considerations

The implementation maintains security best practices:
- **Origin validation**: Messages include origin information for validation
- **Message type validation**: Only processes expected message types
- **Data validation**: Validates message data before processing
- **Error boundary**: Prevents crashes from malformed messages

## Performance Impact

The changes have minimal performance impact:
- **Event listener**: Single global message listener (no change)
- **State management**: Lightweight state object with minimal memory usage
- **Route navigation**: Uses existing Vue Router (no additional overhead)
- **Message processing**: Efficient switch statement for message routing

## Next Steps

1. **Test thoroughly** in development environment
2. **Deploy to staging** for integration testing with parent application
3. **Monitor console logs** for any unexpected behavior
4. **Verify all action types** work correctly with parent application
5. **Test error scenarios** to ensure robust error handling

## Files Modified

1. `src/lib/demo/DemoConstants.ts` - Extended message types and constants
2. `src/lib/demo/index.ts` - Enhanced message handling and added playground functionality
3. `src/lib/demo/playground-test.ts` - Added testing utilities (new file)
4. `IFRAME_COMMUNICATION_FIX.md` - This documentation file (new file)

## Support

If you encounter any issues with the iframe communication:
1. Check browser console for error messages
2. Use the testing utilities to verify message handling
3. Verify the parent application is sending correctly formatted messages
4. Check that the iframe is loaded in demo mode

The fix is designed to be robust and provide clear debugging information to help identify and resolve any remaining issues.