---
title: 'Go Web项目代码模版'
date: 2025-02-14T18:43:45+08:00
draft: false
tags:
  - Golang
categories:
  - Golang
---

在学习`golang`之初我就在网络上搜索过是否`golang`也有像`Java`那种`Spring`框架结构的项目模版。最初看到[project-layout](https://github.com/golang-standards/project-layout)也有很多人在讨论。最终的结论都是`golang`没有什么所谓的`standard`项目结构。

在工作中我也深刻体会到了这点，几乎每个公司，甚至一个公司中不同组，更加甚至一个组中的不同项目的结构都大相径庭。所以我对所谓的`standard`项目结构也就不再执着了。

但是最近在学习中，又接触到重新创建一个项目，于是这次我创建一个适合自己的`golang`项目结构。

其中整体结构参考的是[Let’s Go Further](https://lets-go-further.alexedwards.net/)中的结构组织。我选择这种方式的原因如下：

1. 没有添加其它复杂的第三方依赖，大部分是`golang`自带的库。
2. 结构相对简单，新手容易理解上手

```
.
├── Makefile
├── bin
│   ├── api
│   └── linux_amd64
│       └── api
├── cmd
│   └── api
│       ├── context.go
│       ├── errors.go
│       ├── healthcheck.go
│       ├── helpers.go
│       ├── main.go
│       ├── middleware.go
│       ├── movies.go
│       ├── routes.go
│       ├── server.go
│       ├── tokens.go
│       └── users.go
├── go.mod
├── go.sum
├── internal
│   ├── context
│   │   └── context.go
│   ├── data
│   │   ├── filters.go
│   │   ├── models.go
│   │   ├── movies.go
│   │   ├── permissions.go
│   │   ├── runtime.go
│   │   ├── tokens.go
│   │   ├── tx.go
│   │   └── users.go
│   ├── jsonlog
│   │   └── jsonlog.go
│   ├── mailer
│   │   ├── mailer.go
│   │   └── templates
│   │       ├── token_activation.tmpl
│   │       ├── token_password_reset.tmpl
│   │       └── user_welcome.tmpl
│   ├── service
│   │   ├── service.go
│   │   └── user.go
│   ├── validator
│   │   └── validator.go
│   └── vcs
│       └── vcs.go
├── migrations
│   ├── 000001_create_movies_table.down.sql
│   ├── 000001_create_movies_table.up.sql
└── readme.md
```

但是[Let’s Go Further](https://lets-go-further.alexedwards.net/)中对数据库的操作缺少事务的操作，但是在实际项目中，事务是保证数据一致性的重要方式。在业务中可能需要经常使用。于是我添加了对事务的支持。参考了`threedots`的一些文章。

对于事务操作，主要是对`sql.Tx`的运用，在多个`Repository`中共享1个`sql.Tx`才能在同一个事务中，但是又要保证灵活性。不需要事务的时候又能使用`sql.DB`对数据库进行操作。

所以对事务进行了抽象，提供一个`TxProvider`接口。

```go
type TxProvider interface {
	Transact(txFunc func(models Models) error) error
}

type TransactionProvider struct {
	db *sql.DB
}

func NewTransactionProvider(db *sql.DB) *TransactionProvider {
	return &TransactionProvider{
		db: db,
	}
}

func (p *TransactionProvider) Transact(txFunc func(models Models) error) error {
	return RunInTx(p.db, func(tx *sql.Tx) error {
		models := Models{
			Movies: NewPostgresMovieRepository(tx),
			Users:  NewPostgresUserRepository(tx),
		}

		return txFunc(models)
	})
}

func RunInTx(db *sql.DB, fn func(tx *sql.Tx) error) error {
	tx, err := db.Begin()
	if err != nil {
		return err
	}

	err = fn(tx)
	if err == nil {
		return tx.Commit()
	}

	rollbackErr := tx.Rollback()
	if rollbackErr != nil {
		return errors.Join(err, rollbackErr)
	}

	return err
}
```

通过`TxProvider`接口初始化的`Models`中的Repository都是`*sql.Tx`, 保证了所有的数据操作都是在同一个事务中。

在我们的`Services`中，我们初始化了`data.Models`和`data.TxProvider`。分别表示非事务和事务版本的数据库操作。

```go
type Services struct {
	models     data.Models
	txProvider data.TxProvider
}

func NewServices(models data.Models, txProvider data.TxProvider) Services {
	return Services{
		models:     models,
		txProvider: txProvider,
	}
}

```

最后看一下使用，如果需要事务只需要调用`s.txProvider.Transact`即可。

```go
err = s.txProvider.Transact(func(models data.Models) error {
  err = models.Users.Insert(user)
  if err != nil {
    switch {
    case errors.Is(err, data.ErrDuplicateEmail):
      v.AddError("email", "a user with this email address already exists")
      return nil
    default:
      return err
    }
  }

  err = models.Permissions.AddForUser(user.ID, "movies:read")
  if err != nil {
    return err
  }

  token, err = models.Tokens.New(user.ID, 3*24*time.Hour, data.ScopeActivation)
  if err != nil {
    return err
  }

  return nil
})
```

在这里我从 [Database Transactions in Go with Layered Architecture](https://threedots.tech/post/database-transactions-in-go/)中学习到一点的是。一般开发是从数据库角度出发，1个table对应1个repo。但是这种在处理事务的时候会带来一定的麻烦。虽然可以用上述的`TxProvider`做到。但是我这里是想表达`threedots`提到的一种思考方式。


比如现在的业务是有1个token表和1个user表。现在通过token来激活用户。查询token，找到对应的user，然后更新user的激活状态。最后删除token记录。


你可以使用`TxProvider`完成这个逻辑。

```go
return s.txProvider.Transact(func(models data.Models) error {
		token, err := models.Tokens.GetByToken(data.ScopeActivation, input.TokenPlaintext)
		if err != nil {
			switch {
			case errors.Is(err, data.ErrRecordNotFound):
				v.AddError("token", "invalid or expired activation token")
				return nil
			default:
				return err
			}
		}

		user, err := models.Users.GetByID(token.UserID)
		if err != nil {
			return err
		}

		if user.Activated {
			v.AddError("user", "user has already been activated")
			return nil
		}

		user.Activated = true
		err = models.Users.Update(user)
		if err != nil {
			return err
		}

		err = models.Tokens.DeleteAllForUser(data.ScopeActivation, user.ID)
		if err != nil {
			return err
		}

		return nil
}
```

另外一种思考方式是，因为user和token是关联的，我们可以把token放到user中

```go
type User struct {
	ID        int64     `json:"id"`
	CreatedAt time.Time `json:"created_at"`
	Name      string    `json:"name"`
	Email     string    `json:"email"`
	Password  password  `json:"-"`
	Activated bool      `json:"activated"`
	Version   int       `json:"-"`
	Token     *Token    `json:"-"`
}
```

```go
return s.txProvider.Transact(func(models data.Models) error {
  err := models.Users.UpdateForToken(data.ScopeActivation, input.TokenPlaintext, func(user *data.User) (bool, error) {
    if user.Activated {
      v.AddError("user", "user has already been activated")
      return false, nil
    }

    user.Activated = true

    return true, nil
  })

  if err != nil {
    return err
  }

	// 插入auditlog的日志
  // models.AuditLogRepository.StoreAuditLog(ctx, log)
  return nil
})
```

```go
func (m PostgresUserRepository) UpdateForToken(tokenScope, tokenPlaintext string, updateFn func(user *User) (bool, error)) error {
	// select from db
	ctx, cancel := context.WithTimeout(context.Background(), 3*time.Second)
	defer cancel()

	tokenHash := sha256.Sum256([]byte(tokenPlaintext))

	query := `
		SELECT hash, user_id
		FROM tokens
		WHERE hash = $1 and scope = $2 AND expiry > $3`
	args := []any{tokenHash[:], tokenScope, time.Now()}

	var token Token
	err := m.db.QueryRowContext(ctx, query, args...).Scan(&token.Hash, &token.UserID)
	if err != nil {
		switch {
		case errors.Is(err, sql.ErrNoRows):
			return ErrRecordNotFound
		default:
			return err
		}
	}

	query = `
	SELECT id, created_at, name, email, password_hash, activated, version
	FROM users
	WHERE id = $1`

	var user User
	err = m.db.QueryRowContext(ctx, query, token.UserID).Scan(
		&user.ID,
		&user.CreatedAt,
		&user.Name,
		&user.Email,
		&user.Password.hash,
		&user.Activated,
		&user.Version,
	)
	if err != nil {
		switch {
		case errors.Is(err, sql.ErrNoRows):
			return ErrRecordNotFound
		default:
			return err
		}
	}

	user.Token = &token

	updated, err := updateFn(&user)
	if err != nil {
		return err
	}

	if !updated {
		return nil
	}

	// update in db
	query = `
		UPDATE users
		SET name = $1, email = $2, password_hash = $3, activated = $4, version = version + 1
		WHERE id = $5 AND version = $6
		RETURNING version`

	args = []any{
		user.Name,
		user.Email,
		user.Password.hash,
		user.Activated,
		user.ID,
		user.Version,
	}

	err = m.db.QueryRowContext(ctx, query, args...).Scan(&user.Version)
	if err != nil {
		switch {
		case err.Error() == `pq: duplicate key value violates unique constraint "users_email_key"`:
			return ErrDuplicateEmail
		case errors.Is(err, sql.ErrNoRows):
			return ErrEditConflict
		default:
			return err
		}
	}

	query = `
	DELETE FROM tokens
	WHERE scope = $1 AND user_id = $2`

	_, err = m.db.ExecContext(ctx, query, token, token.UserID)
	if err != nil {
		return err
	}

	return nil
}
```

这种方式是将token和user的操作都在userRepo中完成，通过1个`updateFn`来更新数据。

这2种方式都没啥问题，看个人喜好吧。这2者也并不矛盾，可以同时使用。比如在使用`UpdateForToken`后，需要记录audit日志。audit日志需要单独的repo来处理。同时audit和用户激活的操作需要在同一个事务中，那么在`UpdateForToken`后，继续调用auditRepo来记录日志也是没有问题的。

# 参考资料

- [project-layout](https://github.com/golang-standards/project-layout)
- [Common Anti-Patterns in Go Web Applications](https://threedots.tech/post/common-anti-patterns-in-go-web-applications/)
- [Database Transactions in Go with Layered Architecture](https://threedots.tech/post/database-transactions-in-go/)
- [Let’s Go Further](https://lets-go-further.alexedwards.net/)
