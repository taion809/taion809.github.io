---
date: 2015-12-07T09:00:43-05:00
draft: false
title: basic sqlx relation scanning
description: Really basic relationship scanning with jmoiron/sqlx, I'd rather not forget.
tags: [ "golang" ]
Section: blog
categories:
  - "Development"
slug: basic-sqlx-relation-scanning
---

Very quick, I wanted to use `Get` with a query that contained other tables.  The documentation does not have a lot of examples so here we go.

Given the following database schema

```sql
CREATE EXTENSION pgcrypto;

CREATE TABLE content_types (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    name VARCHAR(255),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    deleted_at TIMESTAMP DEFAULT NULL
);

CREATE TABLE content (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    content_type_id UUID REFERENCES content_types(id) ON DELETE CASCADE ON UPDATE CASCADE,
    content TEXT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    deleted_at TIMESTAMP DEFAULT NULL
)
```

I want to query and return a content row with its content_type, here is the struct I want to return.

```go
package main

import (
	"fmt"
	"github.com/jmoiron/sqlx"
	_ "github.com/lib/pq"
	"log"
	"time"
)

type ContentType struct {
	Id        string     `db:"id"`
	Name      string     `db:"name"`
	CreatedAt *time.Time `db:"created_at"`
	UpdatedAt *time.Time `db:"updated_at"`
	DeletedAt *time.Time `db:"deleted_at"`
}

type Content struct {
	Id          string     `db:"id"`
	Content     string     `db:"content"`
	CreatedAt   *time.Time `db:"created_at"`
	UpdatedAt   *time.Time `db:"updated_at"`
	DeletedAt   *time.Time `db:"deleted_at"`
	ContentType `db:"content_type"`
}

func main() {
	connection, err := sqlx.Connect("postgres", "postgres://user:pass@localhost:5432/database")
	if err != nil {
		log.Fatal(err)
	}

	defer connection.Close()

	query := `
		SELECT content.*,
			   content_types.id AS "content_type.id",
			   content_types.name AS "content_type.name",
			   content_types.created_at AS "content_type.created_at",
			   content_types.updated_at AS "content_type.updated_at"
		FROM content
		JOIN content_types ON content.content_type_id = content_types.id
		WHERE content.id = $1`

	var content Content
	err = connection.Get(&content, query, "UUID-ID-HERE")
	if err != nil {
		log.Fatal(err)
	}

	fmt.Printf("Fetched content: %s of type %s\n", content.Content, content.ContentType.Name)
}

```

I hope this helps :)
