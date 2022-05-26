# Exchange Plugin Interface

## Exchange API Required Functions
### ExchangeOpen
    Called at startup for all exchange plugins. Retrieves the name of the exchange, and sets up two callback functions.

- Parameters:
    - Name (out)

- Returns:
    - Name ?

### ExchangeLogin
    Login or logout to the exchange's API server. Called repeatedly, if the connection to the server was lost, at regular intervals until it is logged in again. Make sure that the function internally detects the login state and returns safely when the user was still logged in. 

- Parameters:
    - User: string
    - Pwd: string
    - IsDemoAccount: boolean

- Returns:
    - LoginState: boolean

### ExchangeStatus
    Returns connection status and optionally the server time. Repeatedly called during the trading session. Can be used for receiving a 'heartbeat'. 

- Parameters:

- Returns:
    - TimeUTC: integer
    - ConnectionState: ["ok", "problem", "market closed"]

- Remarks
    - If the server time is returned, but does not change for several minutes, it is assumed that the exchange server is offline. Trading and other activities are then suspended. 
    - The time format used is the Linux time format, which is the number of seconds since January 1st 1970 midnight: 

### ExchangeAsset
    Subscribes an asset, and/or returns information about it.

- Parameters:
    - Asset

- Returns:

- Remarks

### ExchangeHistory
    Returns the price history of an asset.

- Parameters:
    - Asset

- Returns:

- Remarks

### ExchangeAccount
     Is called in regular intervals and returns the current account status. Is also used to change the account if multiple accounts are supported.

- Parameters:

- Returns:

- Remarks

### ExchangeBuy
    Enters a long or short trade either at market, or at a price limit. Also used for NFA compliant accounts to close a trade by opening a position in the opposite direction. The order type (FOK, IOC, GTC) was set with SET_ORDERTYPE before. Default is FOK ("Fill-Or-Kill"). 

- Parameters:
    - Asset
    - Amount

- Returns:

- Remarks

## Exchange API Optional Functions
### ExchangeTradeState
    Returns the trade state; normally only used for non-NFA compliant exchange APIs that support individual trades. Returns the status of an open or recently closed trade.

- Parameters:

- Returns:

- Remarks

### ExchangeSell
    Closes a trade - completely or partially - at market or at a limit price. If partial closing is not supported by the exchange, the trade is completely closed. Only for not NFA compliant accounts; otherwise the trade is closed by calling ExchangeBuy with the negative amount.  

- Parameters:

- Returns:

- Remarks

### ExchangeCommand
    Sets various plugin parameters or returns asset specific extra data. This function is not mandatory, as it is not used; but it can be called in strategies for special purposes.
    
#### SET_ORDERTYPE
