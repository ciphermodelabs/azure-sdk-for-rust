# azure_identity

Azure Identity crate for the unofficial Microsoft Azure SDK for Rust. This crate is part of a collection of crates: for more information please refer to [https://github.com/azure/azure-sdk-for-rust](https://github.com/azure/azure-sdk-for-rust).
This crate provides mechanisms for several ways to authenticate against Azure

For example, to authenticate using the recommended `DefaultAzureCredential`, you can do the following:

```rust
use azure_core::{auth::TokenCredential, Url};
use azure_identity::{DefaultAzureCredential};

use std::env;
use std::error::Error;

#[tokio::main]
async fn main() -> Result<(), Box<dyn Error>> {
    let credential = DefaultAzureCredential::default();
    let response = credential
        .get_token(&["https://management.azure.com/.default"])
        .await?;

    let subscription_id = env::var("AZURE_SUBSCRIPTION_ID")?;
    let url = Url::parse(&format!(
        "https://management.azure.com/subscriptions/{}/providers/Microsoft.Storage/storageAccounts?api-version=2019-06-01",
        subscription_id))?;
    let response = reqwest::Client::new()
        .get(url)
        .header("Authorization", format!("Bearer {}", response.token.secret()))
        .send()
        .await?
        .text()
        .await?;

    println!("{:?}", response);
    Ok(())
}
```

The supported authentication flows are:
* [Authorization code flow](https://docs.microsoft.com/azure/active-directory/develop/v2-oauth2-auth-code-flow).
* [Client credentials flow](https://docs.microsoft.com/azure/active-directory/develop/v2-oauth2-client-creds-grant-flow).
* [Device code flow](https://docs.microsoft.com/azure/active-directory/develop/v2-oauth2-device-code).

This crate also includes utilities for handling refresh tokens and accessing token credentials from many different sources.

License: MIT
