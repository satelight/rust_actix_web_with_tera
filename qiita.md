


```Cargo.toml
[package]
name = "reqwest-practice"
version = "0.1.0"
edition = "2021"

# See more keys and their definitions at https://doc.rust-lang.org/cargo/reference/manifest.html

[dependencies]
anyhow = "1.0.70"
reqwest = { version = "0.11.16", features = ["blocking"] }
tokio = { version = "1.27.0", features = ["full"] }
```


### 同期処理のみで実装する場合は上記の"blocking"を追記する必要がある。

```main.rs
use reqwest;

fn main() {
    let url = "https://www.rust-lang.org";
    let rc = reqwest::blocking::get(url).unwrap(); //同期処理にするためreqwest::blockingから呼び出す。
    let contents = rc.text().unwrap();
    println!("{}", contents);
}

```


###　公式のドキュメントで説明されている非同期処理の場合
デファクトスタンダードな非同期処理のクレートtokioを利用する必要がある。
その説明がないのでRust初学者の鬼門である。
```main.rs
use reqwest;

#[tokio::main]
async fn main() -> Result<(), Box<dyn std::error::Error>> {
    let url = "https://www.rust-lang.org";
    let contents = reqwest::get(url).await?.text().await?;
    println!("{:?}", contents);
    Ok(())
}

```


### 非同期処理で関数を分割した場合
```
use reqwest;
async fn sub() -> Result<(), Box<dyn std::error::Error>> {
    let url = "https://www.rust-lang.org";
    let contents = reqwest::get(url).await?.text().await?;
    println!("{:?}", contents);
    Ok(())
}

#[tokio::main]
async fn main() -> Result<(), Box<dyn std::error::Error>> {
    sub().await?;
    Ok(())
}
```

### actix_webでreqwestクレートを利用した場合
```
use actix_web::{get, post, web, App, HttpResponse, HttpServer, Responder};

use reqwest;

async fn sub() -> Result<String, reqwest::Error> {
    let url = "https://www.rust-lang.org";
    let contents = reqwest::get(url).await?.text().await?;
    println!("{:?}", contents);
    Ok(contents)
}

#[get("/")]
async fn hello() -> impl Responder {
    sub().await.unwrap();
    HttpResponse::Ok().body("Hello world!")
}

#[post("/echo")]
async fn echo(req_body: String) -> impl Responder {
    HttpResponse::Ok().body(req_body)
}

async fn manual_hello() -> impl Responder {
    HttpResponse::Ok().body("Hey there!")
}

#[actix_web::main]
async fn main() -> std::io::Result<()> {
    // sub().await?;
    HttpServer::new(|| {
        App::new()
            .service(hello)
            .service(echo)
            .route("/hey", web::get().to(manual_hello))
    })
    .bind(("127.0.0.1", 8080))?
    .run()
    .await
}
```