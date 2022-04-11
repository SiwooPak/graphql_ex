### GraphQL 연동 및 사용
#### 설치
```
$ npm i express express-graphql graphql
$ npm i --save-dev nodemon
```
#### 연동
```js
app.use('/graphql', expressGraphQL({
  schema: schema,
  graphiql: true
}))
```

#### 스키마 작성
```js
const BookType = new GraphQLObjectType({
  name: 'Book',
  description: 'This represents a book written by an author',
  fields: () => ({
    id: { type: new GraphQLNonNull(GraphQLInt) },
    name: { type: new GraphQLNonNull(GraphQLString) },
    authorId: { type: new GraphQLNonNull(GraphQLInt) },
    author: {
      type: AuthorType,
      resolve: (book) => {
        return authors.find(author => author.id === book.authorId)
      }
    }
  })
})

```

#### 조회
- 조회는 query
```js
const RootQueryType = new GraphQLObjectType({
  name: 'Query',
  description: 'Root Query',
  fields: () => ({
    book: {
      type: BookType,
      description: 'A Single Book',
      args: {
        id: { type: GraphQLInt }
      },
      resolve: (parent, args) => books.find(book => book.id === args.id)
    },
    books: {
      type: new GraphQLList(BookType),
      description: 'List of All Books',
      resolve: () => books
    },
    authors: {
      type: new GraphQLList(AuthorType),
      description: 'List of All Authors',
      resolve: () => authors
    },
    author: {
      type: AuthorType,
      description: 'A Single Author',
      args: {
        id: { type: GraphQLInt }
      },
      resolve: (parent, args) => authors.find(author => author.id === args.id)
    }
  })
})
```

#### 추가
- 추가 및 수정 및 삭제는 mutataion 
```js
const RootMutationType = new GraphQLObjectType({
  name: 'Mutation',
  description: 'Root Mutation',
  fields: () => ({
    addBook: {
      type: BookType,
      description: 'Add a book',
      args: {
        name: { type: new GraphQLNonNull(GraphQLString) },
        authorId: { type: new GraphQLNonNull(GraphQLInt) }
      },
      resolve: (parent, args) => {
        const book = { id: books.length + 1, name: args.name, authorId: args.authorId }
        books.push(book)
        return book
      }
    },
    addAuthor: {
      type: AuthorType,
      description: 'Add an author',
      args: {
        name: { type: new GraphQLNonNull(GraphQLString) }
      },
      resolve: (parent, args) => {
        const author = { id: authors.length + 1, name: args.name }
        authors.push(author)
        return author
      }
    } 
  })
})
```

#### 스키마 연결
```js
app.use('/graphql', expressGraphQL({
  schema: schema,
  graphiql: true
}))
```

### GraphQL과 RestAPI
- GraphQL: 페이스북에서 만든 어플리케이션 레이어 쿼리 언어
- 기존의 웹, 모바일 앱에서 api를 구현할 때는, 통상적으로 restful api를 사용
- 하지만 restful api를 사용한다면 클라이언트에서 어떠한 기능이 필요할 때마다 
새로운 api를 만들어줘야 했음.
- 그로 인해 기능이 추가될때마다 엔드포인트가 늘어나며 문서화의 부담이 생기게 됨
- GraphQL의 경우 하나의 종단점으로 데이터를 가져올수 있으며,스키마 자체가 문서로서
동작하기에 이점을 갖는다
- 또한 Over-Fetching과 Under-Fetching을 개선해 줌.
#### 단점
- 파일 전송등 텍스트만으로 하기 힘든 작업들을 처리하기 복잡함
- http와 https에 의한 캐싱이 rest보다 복잡함
- 고정된 요청과 응답만 필요할 경우에는 쿼리로 인해 요청의 크기가 rest보다 커짐
- api 유지 관리자의 경우 유지 관리 가능한 graphql 스키마 작성을 위한 추가 태스크를 작성하기
위해 익숙하지 않는 개발자들에겐 다소 작업시간 소요됨
