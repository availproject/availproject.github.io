# Embedding the light client

## When and how to embed the light client

The Avail light client plays a vital role in ensuring the availability and correctness of data within the Avail network. By employing random sampling, it achieves security levels comparable to full nodes. Furthermore, by leveraging the peer-to-peer network, it enhances overall data availability while reducing the load on full nodes.  
The light client is capable of downloading and verifying application-specific data submitted to Avail, which can be conveniently queried using the light client API.  
The light client exposes an HTTP API that enables users to query the status, confidence, and application data for each processed block. When a block is finalized in Avail, the light client performs random sampling and verification, calculates confidence in the given block data, and if the confidence is high, retrieves the application data from the block. This data is then verified and stored locally for easy access.

## Rust examples

### Fetching the number of the latest block processed by light client

To fetch the number of the latest block processed by light client, we can perform `GET` request on `/v1/latest_block` endpoint.


```rust
use reqwest::StatusCode;
use serde::{Deserialize, Serialize};

#[derive(Serialize, Deserialize, Debug, Clone)]
pub struct LatestBlock {
    pub latest_block: u32,
}

const LIGHT_CLIENT_URL: &str = "http://127.0.0.1:7000";

let latest_block_url = format!("{LIGHT_CLIENT_URL}/v1/latest_block");
let response = reqwest::get(latest_block_url).await.unwrap();

if response.status() == StatusCode::OK {
    let latest_block: LatestBlock =
        serde_json::from_str(&response.text().await.unwrap()).unwrap();
    println!("{latest_block:?}");
}
// ...error handling...
```

### Fetching the confidence for given block

To fetch the the confidence for specific block, which is already processed by application client, we can perform `GET` request on `/v1/confidece/{block_number}` endpoint.

```rust
use reqwest::StatusCode;
use serde::{Deserialize, Serialize};

#[derive(Serialize, Deserialize, Debug, Clone)]
pub struct Confidence {
    pub block: u32,
    pub confidence: f64,
    pub serialised_confidence: Option<String>,
}
	
const LIGHT_CLIENT_URL: &str = "http://127.0.0.1:7000";

let block_number = 1;
let confidence_url = format!("{LIGHT_CLIENT_URL}/v1/confidence/{block_number}");
let response = reqwest::get(confidence_url).await.unwrap();

if response.status() == StatusCode::OK {
    let confidence: Confidence =
        serde_json::from_str(&response.text().await.unwrap()).unwrap();
    println!("{confidence:?}");
}
// ...error handling...
```

### Fetching decoded application data for given block

After data is verified, it can be fetched with `GET` request on `/v1/appdata/{block_number}` endpoint, by specifying `decode=true` query parameter.

```rust

use base64::Engine as _;
use reqwest::StatusCode;
use serde::{Deserialize, Serialize};

#[derive(Serialize, Deserialize, Debug, Clone)]
pub struct ExtrinsicsData {
    pub block: u32,
    pub extrinsics: Vec<String>,
}

const LIGHT_CLIENT_URL: &str = "http://127.0.0.1:7000";

let block_number = 2;
let confidence_url = format!("{LIGHT_CLIENT_URL}/v1/appdata/{block_number}?decode=true");
let response = reqwest::get(confidence_url).await.unwrap();

if response.status() == StatusCode::OK {
    let data: ExtrinsicsData =
        serde_json::from_str(&response.text().await.unwrap()).unwrap();

    let decoded_bytes = engine::general_purpose::STANDARD
        .decode(&data.extrinsics[0])
        .unwrap();
    let decoded_string = String::from_utf8(decoded_bytes).unwrap();
    println!("{decoded_string:?}");
}

// ...error handling...
```


### Waiting for application data to be verified

If light client is still processing specific block, we can poll light client with `GET` request on `/v1/appdata/{block_number}` endpoint, and wait for data to became available.

```rust
use avail_subxt::primitives::AppUncheckedExtrinsic;
use serde::{Deserialize, Serialize};
use anyhow::anyhow;
use reqwest::StatusCode;

#[derive(Serialize, Deserialize, Debug, Clone)]
pub struct ExtrinsicsData {
    pub block: u32,
    pub extrinsics: Vec<AppUncheckedExtrinsic>,
}

const POLLING_TIMEOUT: Duration = Duration::from_secs(120);
const POLLING_INTERVAL: Duration = Duration::from_secs(1);
const LIGHT_CLIENT_URL: &str = "http://127.0.0.1:7000";

async fn wait_for_appdata(appdata_url: &str, block: u32) -> anyhow::Result<ExtrinsicsData> {
    let appdata_url = format!("{LIGHT_CLIENT_URL}/v1/appdata/{block}");
    
    let start_time = std::time::Instant::now();

    loop {
        if start_time.elapsed() >= POLLING_TIMEOUT {
            return Err(anyhow!("Poll timeout exceeded"));
        }

        let response = reqwest::get(&appdata_url).await?;

        match response.status() {
            // If status is OK. response continas json representation of submitted data
            StatusCode::OK => {
                let text = &response.text().await?;
                return Ok(serde_json::from_str(text)?);
            }
            // If it is not found, there is no data for given block and application
            StatusCode::NOT_FOUND => {
                return Ok(ExtrinsicsData {
                    block,
                    extrinsics: vec![],
                })
            }
            // Wait for data to became available otherwise
            _ => {
                tokio::time::sleep(POLLING_INTERVAL).await;
                continue;
            }
        }
    }
}

```

Function `wait_for_appdata` will block until data is available or timeout is reached.

## API reference

In case of error, endpoints will return response with `500 Internal Server Error` status code, and descriptive error message.

### **GET** `/v1/mode`

Retrieves the operating mode of the light client. Light client can operate in two different modes, `LightClient` or `AppClient`, depending on configuration of application ID.

#### Responses

If operating mode is `LightClient` response is:

> Status code: `200 OK`
```json
"LightClient"
```

In case of `AppClient` mode, response is:

> Status code: `200 OK`
```json
{"AppClient": {app_id}}
```

### **GET** `/v1/latest_block`

Retrieves the latest block processed by the light client.

#### Responses

> Status code: `200 OK`
```json
{"latest_block":{block_number}}
```

### **GET** `/v1/confidence/{block_number}`

Given a block number, it returns the confidence computed by the light client for that specific block.

> Path parameters:
- `block_number` - block number (requred)

#### Responses

In case when confidence is computed:

> Status code: `200 OK`
```json
{"block":1,"confidence":93.75,"serialised_confidence":"5232467296"}
```

If confidence is not computed, and specified block is before the latest processed block:

> Status code: `400 Bad Request`
```json
"Not synced"
```

If confidence is not computed, and specified block is after the latest processed block:

> Status code: `404 Not Found`
```json
"Not found"
```

### **GET** `/v1/appdata/{block_number}`

Given a block number, it retrieves the hex-encoded extrinsics for the specified block, if available. Alternatively, if specified by a query parameter, the retrieved extrinsic is decoded and returned as a base64-encoded string.

> Path parameters:
- `block_number` - block number (requred)

> Query parameters:
- `decode` - `true` if decoded extrinsics is requested (boolean, optional, default is `false`)

#### Responses

If application data is available, and decode is `false` or unspecified:

> Status code: `200 OK`
```json
{"block":1,"extrinsics":["0xc5018400d43593c715fdd31c61141abd04a99fd6822c8558854ccde39a5684e7a56da27d01308e88ca257b65514b7b44fc1913a6a9af6abc34c3d22761b0e425674d68df7de26be1c8533a7bbd01fdb3a8daa5af77df6d3fb0a67cde8241f461f4fe16f188000000041d011c6578616d706c65"]}
```

If application data is available, and decode is `true`:

> Status code: `200 OK`
```json
{"block":1,"extrinsics":["ZXhhbXBsZQ=="]}
```

If application data is not available, and specified block is the latest block:

> Status code: `401 Unauthorized`
```json
"Processing block"
```

If application data is not available, and specified block is not the latest block:

> Status code: `404 Not Found`
```json
"Not found"
```

### **GET** `/v1/status/{block_number}`

Retrieves the status of the latest block processed by the light client.

> Path parameters:
- `block_number` - block number (requred)

#### Responses

If latest processed block exists, and `app_id` is configured (otherwise, `app_id` is not set):

> Status code: `200 OK`
```json
{"block_num":89,"confidence":93.75,"app_id":1}
```

If there are no processed blocks:

> Status code: `404 Not Found`
```json
"Not found"
```
