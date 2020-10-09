up-api-haskell
==============

A Haskell wrapper for the Up Bank API ⚡.

### Usage

```Haskell
import Data.Aeson.Encode.Pretty
import Network.HTTP.Client (newManager)
import Network.HTTP.Client.TLS (tlsManagerSettings)
import Servant.Client
import System.Environment

import qualified Data.ByteString.Lazy.Char8 as BLC

import Up

import Up.API
import Up.Model.Account
import Up.Model.Paginated
import Up.Model.Transaction
import Up.Model.Token


-- | Query the Up API
query :: ClientM (Account, [Transaction])
query = do
    -- An account
    account <- head <$> paginatedData <$> listAccounts Nothing
    let aid = accountId $ account
    -- A list of transactions from account (max 2)
    transactions <- paginatedData <$> listTransactionsByAccount aid (Just 2) Nothing Nothing Nothing
    pure (account, transactions)

run :: IO ()
run = do
    -- Get the API token from the system environment variables
    token <- Token <$> getEnv "UP_BANK_TOKEN"
    mgr <- newManager tlsManagerSettings

    -- Run the queries
    res <- runClientM query (mkClientEnv mgr upBaseUrl) 
        { makeClientRequest = makeUpClientRequest token}

    case res of
      Left err -> putStrLn $ show err
      Right u -> BLC.putStrLn $ encodePretty u
```
