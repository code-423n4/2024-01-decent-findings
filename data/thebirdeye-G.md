1. Instead of using address(this). It is more gas efficient to pre calculate and use hard coded address.
2. with assembly .call(bool success) is more gas optimized than (bool success).
3. Use "call" instead of "transfer" to send Ether. It is the recommended way of sending Ether in a smart contract.