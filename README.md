# Rust code generation for sqlx CRUD

**This is a prototype!** There's nothing robust about this yet and should not be relied upon.

This inspects a database (tested on Postgres) and generates structs and functions to query the tables in the database.

Using the following SQL:

```sql
CREATE TABLE app_user (
	id bigserial PRIMARY KEY,
	name text NOT NULL,
	email text NOT NULL
);
```

The following code is currently generated (after rustfmt):
```rust
pub struct AppUserRow {
    pub id: i64,
    pub name: String,
    pub email: String,
}

pub struct AppUserInputRow {
    pub name: String,
    pub email: String,
}

pub async fn insert_app_user(
    conn: &mut PgConnection,
    row: &AppUserInputRow,
) -> Result<AppUserRow, sqlx::Error> {
    let result = sqlx::query_as!(
        AppUserRow,
        r#"INSERT INTO app_user ("name", "email")
            VALUES ($1, $2) RETURNING *"#,
        row.name,
        row.email
    )
    .fetch_one(&mut *conn)
    .await?;
    Ok(result)
}

pub async fn select_app_user(conn: &mut PgConnection, id: &i64) -> Result<AppUserRow, sqlx::Error> {
    let result = sqlx::query_as!(
        AppUserRow,
        r#"SELECT "id", "name", "email" FROM app_user WHERE id=$1"#,
        id
    )
    .fetch_one(&mut *conn)
    .await?;
    Ok(result)
}
```

Standard environment variables like `DATABASE_URL`, `PGUSER`, and `PGPASSWORD` may be used to configure the generator.

To run, clone this repo and run `cargo run` with the correct environment variables set. Code is emitted to stdout.
