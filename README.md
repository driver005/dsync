# dsync

<a href="https://crates.io/crates/dsync"><img src="https://img.shields.io/crates/v/dsync.svg?style=for-the-badge" height="20" alt="License: MIT OR Apache-2.0" /></a>

A utility to generate database structs and querying code from diesel schema files. Primarily built for [create-rust-app](https://github.com/Wulf/create-rust-app).

Currently, it's more advantageous to generate code over deriving code with macros because intellisense and autocompletion isn't quite there when it comes to macro expansion.

## Demo

Given the following schema:

```rust
// schema.rs
diesel::table! {
    todos (id) {
        id -> Int4,
        text -> Text,
        completed -> Bool,
    }
}
```

We run:

```sh
cargo dsync -i schema.rs -o models
```

Now we have everything we need!

```rust
use models::todos;

async fn demo(db: Connection) {
  let created_todo = todos::create(&mut db, todos::CreateTodo {
    text: "Create a demo",
    completed: false,
  }).await?;
  
  let todos_list = todos::paginate(&mut db, 1, 10).await?;
  
  let updated_todo = todos::update(&mut db, created_todo.id, UpdateTodo {
    text: created_todo.text,
    completed: true,
  }).await?;
}
```

For a complete example, see [`test/simple_table/schema.rs`](test/simple_table/schema.rs) which generates all the code in [`test/simple_schema/models`](test/simple_table/models).

## Usage

1. Add this crate:

    ```sh
    cargo add dsync
    ```

2. Create a new binary in your project which uses the crate (for example, `bin/dsync.rs`)

   ```rust
   use std::{collections::HashMap, path::PathBuf};
   use dsync::{GenerationConfig, TableOptions};
   
   pub fn main() {
       let dir = env!("CARGO_MANIFEST_DIR");
   
       dsync::generate_files(
           PathBuf::from_iter([dir, "src/schema.rs"]), 
           PathBuf::from_iter([dir, "src/models"]), 
           GenerationConfig { /* ... your generation options ... */ }
       );
   }
   ```

3. Create a `Cargo.toml` binary entry:

   ```toml
   [[bin]]
   name = "dsync"
   path = "bin/dsync.rs"
   ```

4. Execute!

  ```sh
  cargo run --bin dsync
  ```

  **Protip**: to use `cargo dsync`, create an alias in `.cargo/config`:
  
  ```toml
  [alias]
  dsync="run --bin dsync"
  ```

### Pre-built binary

Setting up a custom binary allows you to completely customize the generation; however, if complete customization isn't necessary, you can install the CLI directly
(you'll have to make sure you keep it up-to-date by running this periodically):

```sh
cargo install dsync 
```

**CLI Usage**

* `-i`: input argument: path to schema file
* `-o`: output argument: path to directory where generated code should be written
* `-c`: connection type (for example: `diesel::r2d2::PooledConnection<diesel::r2d2::ConnectionManager<diesel::PgConnection>>`)  
* `-g`: (optional) list of columns that are automatically generated by create/update triggers (for example, `created_at`, `updated_at`)
* `--tsync`: (optional) adds `#[tsync]` attribute to generated structs (see <https://github.com/Wulf/tsync>)
* `--model-path`: (optional) set a custom model import path, default `crate::models::`
* `--schema-path`: (optional) set a custom schema import path, default `crate::schema::`
* `--no-serde`: (optional) if set, does not output any serde related code
* `--no-crud`: (optional) Do not generate the CRUD functions for generated models
* `--create-str`: (optional) Set which string type to use for `Create*` structs (possible are `string`, `str`, `cow`)
* `--update-str`: (optional) Set which string type to use for `Update*` structs (possible are `string`, `str`, `cow`)
* `--single-model-file`: (optional) Generate only a single model file, instead of a directory with `mod.rs` and `generated.rs`
* note: the CLI has fail-safes to prevent accidental file overwriting

```sh
dsync -i src/schema.rs -o src/models
```

## Docs

See `dsync --help` for more information.

Feel free to open tickets for support or feature requests.

## Development/Testing

Use `./test/test_all.sh` to run tests.
After running the test, there should be no unexpected changes to files in `./test` (use `git status` and `git diff` to see if there were any changes).

## License

This tool is distributed under the terms of both the MIT license and the Apache License (Version 2.0).

See LICENSE-APACHE, LICENSE-MIT, and COPYRIGHT for details.
