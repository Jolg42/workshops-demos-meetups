# Wroclaw Node Meetup #8 👋

### Typesafe String-References

```ts
objectType({
  name: 'User',
  definition() {
    t.field('comments', {
      type: 'Comment',
      //     ^^^^^^^
    })
  },
})
```

### Typesafe Resolver Param "parent"

```ts
objectType({
  name: 'User',
  definition() {
    t.field('comments', {
      type: 'Comment',
      list: true,
      resolve(user) {
        //    ^^^^
      },
    })
  },
})
```

### Typesafe Resolver Param "args"

```ts
mutationField((t) => {
  t.field('registerUser', {
    type: 'User',
    args: {
      email: stringArg({ required: true }),
      handle: stringArg({ required: true }),
    },
    resolve(_, args) {
      //       ^^^^
    },
  })
})
```

### Typesafe Resolver Return

```ts
objectType({
  name: 'User',
  definition() {
    t.field('comments', {
      type: 'Comment',
      list: true,
      resolve(user, _, ctx) {
        return ctx.db.fetchComments({ userId: user.id })
        //     ^^^^^^^^^^^^^^^^^^^^ result must type check
      },
    })
  },
})
```

### Typesafe Abstract Type Strategies

```ts
unionType({
  name: 'SearchResult',
  definition(t) {
    t.members('User', 'Comment')
  },
})
```

#### Modular

```ts
objectType({
  name: 'User',
  // Type Error: Missing isTypeOf implementation ...
})
```

```ts
objectType({
  name: 'User',
  isTypeOf(data) {
    //     ^^^^ Union of model types
    return true
    //     ^^^^ or false
  },
})
```

#### Centralized

```ts
unionType({
  name: 'SearchResult',
  // Type Error: Missing resolveType implementation ...
  definition(t) {
    t.members('User', 'Comment')
  },
})
```

```ts
unionType({
  name: 'SearchResult',
  resolveType(data) {
    //        ^^^^ Union of model types
    return 'User'
    //      ^^^^ or 'Comment'
  },
  definition(t) {
    t.members('User', 'Comment')
  },
})
```

#### Discriminant Model Field

```ts
queryType({
  definition(t) {
    t.field('search', {
      type: 'SearchResult',
      args: {
        pattern: stringArg(),
      },
      resolve(_, args, ctx) {
        const result = ctx.db.search(args.pattern)
        return result
        //     ^^^^^^ Type Error: missing __typename
      },
    })
  },
})
```

### Plugins

#### Model Field Projection

```ts
objectType({
  name: 'User',
  definition(t) {
    t.model.id()
    t.model.email()
    t.model.handle()
    t.model.comments({ pagination: true })
  },
})
```

#### Automated CRUD

```ts
mutationType({
  definition(t) {
    t.crud.createUser()
    t.crud.updateUser()
    t.crud.deleteUser()
  },
})
```

#### Relay Connections

```ts
objectType({
  name: 'User',
  definition(t) {
    t.connection('comments')
  },
})
```

#### Field Authorization

```ts
queryType({
  definition(t) {
    t.field('post', {
      type: 'Post',
      args: {
        id: idArg({ required: true }),
      },
      authorize(root, args, ctx) {
        return ctx.auth.canViewPost(args.id)
      },
      resolve(root, args, ctx) {
        return ctx.post.byId(args.id)
      },
    })
  },
})
```
