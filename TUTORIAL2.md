# Welcome to Redwood, Part II: Redwood's Revenge

Part 1 of the tutorial was a huge success! It introduced hundreds (maybe thousands?) of developers to what Redwood could do to make web development in the Javascript ecosystem a delight. But that was just the beginning.

If you read the README [closely](https://github.com/redwoodjs/redwood#technologies) you may have seen a few technologies that we didn't touch on at all in the first tutorial: [Storybook](https://storybook.js.org/) and [Jest](https://jestjs.io/). In reality, these have been core to the very idea of Redwood from the beginning—an improvement to the entire experience of developing a web application.

While they're totally optional, we believe using these two tools will greatly improve your development experience, making your applications easier to develop, easier to maintain, and easier to share with a larger team. In this second tutorial we're going to show you how.

Oh, and while we're at we'll introduce Role-based Authorization Control (RBAC), which wasn't available when we wrote the first tutorial, but is now, and it's amazing.

## Prerequisites

We highly recommend going through the first tutorial first, or at least have built a slightly complex Redwood app on your own. You've hopefully got experience with:

* Authorization
* Cells
* GraphQL & SDLs
* Services

If you haven't been through the first tutorial, or maybe you went through it on an older version of Redwood (before 0.19.0) you can clone this repo which contains everything built in part 1 and also adds a little styling so it isn't quite so...ugly. Don't get us wrong, what we built in part 1 had a great personality! We just gave it some hipper clothes and a nice haircut. We used [TailwindCSS](https://tailwindcss.com) to style things up and added a `<div>` or two to give us some additional hooks to hang some styling on.

    git clone https://github.com/redwoodjs/redwood-tutorial
    cd redwood-tutorial
    yarn install
    yarn rw db up
    yarn rw db seed
    yarn rw dev

That'll check out the repo, install all the dependencies, create your local database and fill it with a few blog posts, and finally start up the dev server. Your browser should open to a fresh new blog app:

![image](https://user-images.githubusercontent.com/300/95521547-a5f8b000-097e-11eb-911c-5fde4bed6d97.png)

Let's run the test suite to make sure everything is working as expected:

```terminal
yarn rw test
```

This command starts a persistent process which watches for file changes and automatically runs any tests associated with the changed file(s) (changing a component *or* its tests will trigger a test run).

Since we just started the suite, and we haven't changed any files yet, it may not actually run any tests at all. Hit `a` to tell it run **a**ll tests and we should get a passing suite:

![image](https://user-images.githubusercontent.com/300/96655360-21991c00-12f2-11eb-9394-c34c39b69f01.png)

More on testing later, but for now just know that this is always what we want to aim for—all green! In fact best pracitices tell us you should not even commit any code unless the test suite passes locally. Not everyone adhears to this quite as strictly as others...*&lt;cough, cough&gt;*

## Introduction to Storybook

Let's see what this Storybook thing is all about. Run this command to start up the Storybook server:

    yarn rw storybook

After some compling you should get a message saying that Storybook has started and it's available at http://localhost:7910

![image](https://user-images.githubusercontent.com/300/95522673-8f078d00-0981-11eb-9551-0a211c726802.png)

If you poke around at the file tree on the left you'll see all of the components, cells, layouts and pages we created during the tutorial. Where did they come from? You may recall that everytime we generated a new page/cell/component we actaully created at least *three* files:

* BlogPost.js
* BlogPost.stories.js
* BlogPost.test.js

> If you generated a cell then you also got a `.mock.js` file (more on those later).

Those `.stories.js` files are what makes the tree on the left side of the Storybook browser possible! From their homepage, Storybook describes itself as:

*"...an open source tool for developing UI components in isolation for React, Vue, Angular, and more. It makes building stunning UIs organized and efficient."*

So, the idea here is that you can build out your components/cells/pages in isolation, get them looking the way you want and displaying the correct data, then plug them into your full application.

When Storybook opened it should have opened **Components > BlogPost > Generated** which is the generated component we created to display a single blog post. If you open `web/src/components/BlogPost/BlogPost.stories.js` you'll see what it takes to explain this component to Storybook, and it isn't much:

```javascript
import BlogPost from './BlogPost'

export const generated = () => {
  return (
    <BlogPost
      post={{
        id: 1,
        title: 'First Post',
        body: `Neutra tacos hot chicken prism raw denim, put a bird on it enamel pin
              post-ironic vape cred DIY. Street art next level umami squid. Hammock
              hexagon glossier 8-bit banjo. Neutra la croix mixtape echo park four
              loko semiotics kitsch forage chambray. Semiotics salvia selfies jianbing
              hella shaman. Letterpress helvetica vaporware cronut, shaman butcher
              YOLO poke fixie hoodie gentrify woke heirloom.`,
        createdAt: '2020-01-01T12:34:45Z'
      }}
    />
  )
}

export default { title: 'Components/BlogPost' }
```

You import the component you want to use and then all of the named exports in the file will be a single "story" as displayed in Storybook. In this case the generator named it "generated" which shows as the "Generated" story in the tree view:

```terminal
Components
└── BlogPost
    └── Generated
```

This makes it easy to create variants of your component and have them all displayed together.

> Where did that sample blog post data come from? We (the Redwood team) added that to the story in the `redwood-tutorial` repo to show you what a story might look like after you hook up some sample data. Several of the stories need data like this, some inline and some in those `mock.js` files. The rest of the tutorial will be showing you how to do this yourself with new components as you create them.

## Our First Story

Let's say that on our homepage we only want to show the first couple of sentences in our blog post and you'll have to click through to see the full post.

First let's update the `BlogPost` component to contain that functionality:

```javascript{5-7,9,18}
// web/src/components/BlogPost/BlogPost.js

import { Link, routes } from '@redwoodjs/router'

const truncate = (text, length) => {
  return text.substring(0, length) + '...'
}

const BlogPost = ({ post, summary = false }) => {
  return (
    <article className="mt-10">
      <header>
        <h2 className="text-xl text-blue-700 font-semibold">
          <Link to={routes.blogPost({ id: post.id })}>{post.title}</Link>
        </h2>
      </header>
      <div className="mt-2 text-gray-900 font-light">
        {summary ? truncate(post.body, 100) : post.body}
      </div>
    </article>
  )
}

export default BlogPost
```

We'll pass an additional `summary` prop to the component to let it know if should show just the summary or the whole thing. We default it to `false` to preserve the existing behavior—always showing the full body.

Now in the story, let's create a `summary` story that uses `BlogPost` the same way that `generated` does, but adds the new prop. We'll take the content of the sample post and put that in a constant that both stories will use. We'll also rename `generated` to `full` to make it clear what's different between the two:

```javascript{5-14,16-18,20-22}
// web/components/BlogPost/BlogPost.stories.js

import BlogPost from './BlogPost'

const POST = {
  id: 1,
  title: 'First Post',
  body: `Neutra tacos hot chicken prism raw denim, put a bird on it enamel pin
         post-ironic vape cred DIY. Street art next level umami squid. Hammock
         hexagon glossier 8-bit banjo. Neutra la croix mixtape echo park four
         loko semiotics kitsch forage chambray. Semiotics salvia selfies jianbing
         hella shaman. Letterpress helvetica vaporware cronut, shaman butcher
         YOLO poke fixie hoodie gentrify woke heirloom.`,
}

export const full = () => {
  return <BlogPost post={POST} />
}

export const summary = () => {
  return <BlogPost post={POST} summary={true} />
}

export default { title: 'Components/BlogPost' }
```

As soon as you save the change the stories Storybook should refresh and show the updates:

![image](https://user-images.githubusercontent.com/300/95523957-ed823a80-0984-11eb-9572-31f1c249cb6b.png)

### Displaying the Summary

Great! Now to complete the picture let's use the summary in our home page display of blog posts. The actual Home page isn't what references the `BlogPost` component though, that's in the `BlogPostsCell`. We'll add the summary prop and then check the result in Storybook:

```javascript{27}
// web/src/components/BlogPostsCell/BlogPostsCell.js

import BlogPost from 'src/components/BlogPost'

export const QUERY = gql`
  query BlogPostsQuery {
    posts {
      id
      title
      body
      createdAt
    }
  }
`

export const Loading = () => <div>Loading...</div>

export const Empty = () => <div>Empty</div>

export const Failure = ({ error }) => <div>Error: {error.message}</div>

export const Success = ({ posts }) => {
  return (
    <div className="-mt-10">
      {posts.map((post) => (
        <div key={post.id} className="mt-10">
          <BlogPost key={post.id} post={post} summary={true} />
        </div>
      ))}
    </div>
  )
}
```

![image](https://user-images.githubusercontent.com/300/95525432-f4ab4780-0988-11eb-9e9b-8df6641452ec.png)

And if you head to the real site you'll see the summary there as well:

![image](https://user-images.githubusercontent.com/300/95527363-ef9ac800-0989-11eb-9c53-6dc8ab58799c.png)

Storybook makes it easy to create and modify your components in isolation and actually helps enforce a general best practice when building React applications: components should be self-contained and reusable by just changing the props that are sent in.

## Our First Test

So if Storybook is the first phase of creating/updating a component, phase two must be confirming the functionality with a test. Let's add a test for our new summary feature.

First let's run the existing suite to see if we broke anything:

```terminal
yarn rw test
```

Well that didn't take long! Can you guess what we broke?

![image](https://user-images.githubusercontent.com/300/96655765-1b576f80-12f3-11eb-9e92-0024c19703cc.png)

The test was looking for the full text of the blog post, but remember that in `BlogPostsCell` we had `BlogPost` only display the `summary` of the post, not the full text. This test is looking for the full text match.

Let's update the test so that it checks for the expected behavior instead. There are entire books written on the best way to test your code, so no matter how we decide on testing this code there will be someone out there to tell us we're doing it wrong. As just one example: the simplest test would be to just copy what's output and use that for the text in the test:

```javascript
// web/src/components/BlogPostsCell.test.js

test('Success renders successfully', async () => {
  const posts = standard().posts
  render(<Success posts={posts} />)

  expect(screen.getByText(posts[0].title)).toBeInTheDocument()
  expect(screen.getByText("Neutra tacos hot chicken prism raw denim, put a bird on it enamel pin post-ironic vape cred DIY. Str...")).toBeInTheDocument()
})
```

But the number of characters we truncate to could be changed, so how do we encapsulate that in our test? Or should we? The number of characters is in the `BlogPost` component, which this one shouldn't know about. Even if we refactored the `truncate` function into a shared place and imported it into both `BlogPost` and this test, the test will still be knowing too much about `BlogPost`—why should it know the internals of `BlogPost` and that it's making use of this `truncate` function at all? It shouldn't!

Let's compromise—by virtue of the fact that this functionality has a prop called "summary" we can guess that it's doing "something" to shorten the text. So what if we test three things that we can make reasonable assumptions about right now:

1. The full body of the post body *is not* present
2. But, at least the first couple of words of the post *are* present
3. The text that is shown ends in `...`

This gives us a buffer if we decide to truncate to something like 25 words, or even if we go up to a couple of hundred. What it *doesn't* encompass, however, is the case where the body of the blog post is shorter than the truncate limit. In that case the full text would be present, and we should probably update the `truncate` function to not add the `...` in that case. We'll leave adding that functionality and test case up to you in your free time. ;)

### Adding the Test

Okay, let's do this:

```javascript{27-34}
// web/src/components/BlogPostsCell.test.js

import { render, screen } from '@redwoodjs/testing'
import { Loading, Empty, Failure, Success } from './BlogPostsCell'
import { standard } from './BlogPostsCell.mock'

describe('BlogPostsCell', () => {
  test('Loading renders successfully', () => {
    render(<Loading />)
    expect(screen.getByText('Loading...')).toBeInTheDocument()
  })

  test('Empty renders successfully', async () => {
    render(<Empty />)
    expect(screen.getByText('Empty')).toBeInTheDocument()
  })

  test('Failure renders successfully', async () => {
    render(<Failure error={new Error('Oh no')} />)
    expect(screen.getByText(/Oh no/i)).toBeInTheDocument()
  })

  test('Success renders successfully', async () => {
    const posts = standard().posts
    render(<Success posts={posts} />)

    posts.forEach((post) => {
      const truncatedBody = posts[0].body.substring(0, 10)
      const regex = new RegExp(`${truncatedBody}.*?\.{3}`)

      expect(screen.getByText(post.title)).toBeInTheDocument()
      expect(screen.queryByText(post.body)).not.toBeInTheDocument()
      expect(screen.getByText(regex)).toBeInTheDocument()
    })
  })
})
```

This loops through each post in our `standard()` mock and for each one:

`const truncatedBody = posts[0].body.substring(0, 10)`
: Create a variable `truncatedBody` containing the first 10 characters of the post body

``const regex = new RegExp(`${truncatedBody}.*?\.{3}`)``
: Create a regular expression which contains those 10 characters followed by any characters `.*?` until it reaches three periods `\.{3}` (the ellipsis at the end of the truncated text)

`expect(screen.getByText(post.title)).toBeInTheDocument()`
: Find the title in the page

`expect(screen.queryByText(post.body)).not.toBeInTheDocument()`
: When trying to find the *full* text of the body, it should not be present

`expect(screen.getByText(regex)).toBeInTheDocument()`
: Find the truncated-body-plus-ellipsis somewhere in the page

As soon as you saved that test file the test should have run and passed! Press `a` to run the whole suite.

> **What's the difference between `getByText()` and `queryByText()`?**
>
> `getByText()` will throw an error if the text isn't found in the document, whereas `queryByText()` will  return `null`. You can read more about these in the [DOM Testing Library Queries](https://testing-library.com/docs/dom-testing-library/api-queries) docs.

To double check that we're testing what we think we're testing, open up `BlogPostCell.js` and remove the `summary={true}` prop (or set it to `false`)—the test will now fail (because the full body of the post is now on the page and `expect(screen.queryByText(post.body)).not.toBeInTheDocument()` *is* in the document. Make sure to put the `summary={true}` back before we continue!

### Testing BlogPost

Our test suite is passing again but it's a trick! We never added a test for the actual `summary` functionality that we added to the `BlogPost` component! We tested that `BlogPostCell` requests that `BlogPost` return a summary, but what it means to render a summary is knowledge that only `BlogPost` contains.

When you get into the flow of building your app it can be very easy to overlook testing functionality like this. Wasn't it Winston Chuchill who said "a thorough test suite requires eternal vigilence"? Techniques like [Test Driven Development](https://en.wikipedia.org/wiki/Test-driven_development) (TDD) were established to help combat this tendency—write the test first, watch it fail, then write the code to make the test pass. What we're doing is affectionately known as [Development Driven Testing](https://medium.com/table-xi/development-driven-testing-673d3959dac2). You'll probably settle somewhere in the middle but one maxim is always true—some tests are better than no tests.

The summary functionality in `BlogPost` is pretty simple, but there are a couple of different ways we could test it:

* Export the `truncate` function and test it directly
* Test the final rendered state of the component

In this case `truncate` "belongs to" `BlogPost` and the outside world really shouldn't need to worry about it or know that it exists. If we came to a point in development where another component needed to truncate text that would be a perfect time to move this function to a shared location and import it into both components that need it. `truncate` could then have its own dedicated test. But for now let's keep our separation of concerns and test the one thing that's "public" about this component—the result of the render.

In this case let's just test that the output matches an exact string. You could spin yourself in circles trying to refactor the code to make it absolutely bulletproof to code changes breaking the tests, but will you ever actually need that level of flexibility? It's always a trade-off!

We'll move the sample input data to a constant and then use it in both the existing test (which tests that not passing the `summary` prop at all results in the full body being rendered) and our new test that tests for the summary version being rendered:

```javascript{7-12,22-31}
// web/src/components/BlogPost.test.js

import { render, screen } from '@redwoodjs/testing'

import BlogPost, { SUMMARY_LENGTH, SUMMARY_SUFFIX } from './BlogPost'

const POST = {
  id: 1,
  title: 'First post',
  body: `Neutra tacos hot chicken prism raw denim, put a bird on it enamel pin post-ironic vape cred DIY. Street art next level umami squid. Hammock hexagon glossier 8-bit banjo. Neutra la croix mixtape echo park four loko semiotics kitsch forage chambray. Semiotics salvia selfies jianbing hella shaman. Letterpress helvetica vaporware cronut, shaman butcher YOLO poke fixie hoodie gentrify woke heirloom.`,
  createdAt: new Date().toISOString(),
}

describe('BlogPost', () => {
  it('renders a blog post', () => {
    render(<BlogPost post={POST} />)

    expect(screen.getByText(POST.title)).toBeInTheDocument()
    expect(screen.getByText(POST.body)).toBeInTheDocument()
  })

  it('renders a summary of a blog post', () => {
    render(<BlogPost post={POST} summary={true} />)

    expect(screen.getByText(POST.title)).toBeInTheDocument()
    expect(
      screen.getByText(
        'Neutra tacos hot chicken prism raw denim, put a bird on it enamel pin post-ironic vape cred DIY. Str...'
      )
    ).toBeInTheDocument()
  })
})
```

Saving that change should run the tests for that single file and pressing `a` will run them all to make sure the rest of the suite is still happy.

### One Last Thing

Remember we set the `summary` prop to default to `false` if it doesn't exist, which is tested by the first test case. However, we don't have a test that checks what happens if `false` is set explictly. Feel free to add that now if you want Complete Coverage&trade;!

## Building a Component the Redwood Way

What's our blog missing? Comments. Let's add a simple comment engine so people can leave
their completely rational, well-reasoned comments on our blog posts. It's the Internet,
what could go wrong?

There are a couple of ways we could go about building this new feature:

1. Start with the form and then the comment display
2. Start with the comment display and then add the form

To keep things simple let's start with the display first, then we'll move on to more complex work of a form and service to save data.

### Storybook

Let's create a component for the display of a single comment. First up, the generator:

```terminal
yarn rw g component Comment
```

Storybook should refresh and our "Generated" Comment story will be ready to go:

![image](https://user-images.githubusercontent.com/300/95784041-e9596400-0c87-11eb-9b9f-016e0264e0e1.png)

Let's think about what we want to ask users for and then display in a comment. How about just their name and the content of the comment itself? And we'll throw in the date/time it was created. Let's update the Comment component to accept a `comment` object with those two properties:

```javascript{3,6-7}
// web/src/components/Comment/Comment.js

const Comment = ({ comment }) => {
  return (
    <div>
      <h2>{comment.name}</h2>
      <time datetime={comment.createdAt}>{comment.createdAt}</time>
      <p>{comment.body}</p>
    </div>
  )
}

export default Comment
```

Once you save that file and Storybook reloads you'll see it blow up:

![image](https://user-images.githubusercontent.com/300/95784285-6684d900-0c88-11eb-9380-743079870147.png)

We need to update the story to include that comment object and pass it as a prop:

```javascript{8-11}
// web/src/components/Comment/Comment.stories.js

import Comment from './Comment'

export const generated = () => {
  return (
    <Comment
      comment={{
        name: 'Rob Cameron',
        body: 'This is the first comment!',
        createdAt: '2020-01-01T12:34:56Z'
      }}
    />
  )
}

export default { title: 'Components/Comment' }
```

> Note that Datetimes will come from GraphQL in ISO8601 format

Storybook will reload and be much happier:

![image](https://user-images.githubusercontent.com/300/95785006-ccbe2b80-0c89-11eb-8d3b-bdf5ad5a6d63.png)

Let's add a little bit of styling and date conversion to get this Comment component looking like a nice, completed design element:

```javascript{3-7,11-18}
// web/src/components/Comment/Comment.js

const formattedDate = (datetime) => {
  const parsedDate = new Date(datetime)
  const month = parsedDate.toLocaleString('default', { month: 'long' })
  return `${parsedDate.getDate()} ${month} ${parsedDate.getFullYear()}`
}

const Comment = ({ comment }) => {
  return (
    <div className="bg-gray-200 p-8 rounded-lg">
      <header className="flex justify-between">
        <h2 className="font-semibold text-gray-700">{comment.name}</h2>
        <time className="text-xs text-gray-500" dateTime={comment.createdAt}>
          {formattedDate(comment.createdAt)}
        </time>
      </header>
      <p className="text-sm mt-2">{comment.body}</p>
    </div>
  )
}

export default Comment
```

![image](https://user-images.githubusercontent.com/300/95786526-9afa9400-0c8c-11eb-9d75-27c996ca018a.png)

It's tough to see our rounded corners, but rather than adding margin or padding to the component itself (which would add them everywhere we use the component) let's add a margin in the story so it only shows in Storybook:

```javascript{7,15}
// web/src/components/Comment/Comment.stories.js

import Comment from './Comment'

export const generated = () => {
  return (
    <div className="m-4">
      <Comment
        comment={{
          name: 'Rob Cameron',
          body: 'This is the first comment!',
          createdAt: '2020-01-01T12:34:56Z',
        }}
      />
    </div>
  )
}

export default { title: 'Components/Comment' }
```

> A best practice to keep in mind when designing in HTML and CSS is to keep a visual element responsible for its own display only, and not assume what it will be contained within. In this case a Comment doesn't and shouldn't know where it will be displayed, so it shouldn't add any design influence *outside* of its container (like forcing a margin around itself).

Now we can see our roundedness quite easily in Storybook:

![image](https://user-images.githubusercontent.com/300/95786006-aac5a880-0c8b-11eb-86d5-105a3b929347.png)

> If you haven't used TailwindCSS before just know that the `m` in the className is short for "margin" and the `4` refers to four "units" of margin. By default one unit is 0.25rem. So "m-4" is equivalent to `margin: 1rem`.

Our amazing blog posts will obviously garner a huge and passionate fanbase and we will very rarely have only a single comment. Let's work on displaying a list of comments.

### Testing

We don't want Santa to skip our house for being naughty developers so let's test our Comment component. We could test that the author's name and the body of the comment appear, as well as the date it was posted.

The default test that comes with a generated component just makes sure that no errors are thrown, which is the least we could ask of our components!

Let's add a sample Comment to the test and check that the various parts are being rendered:

```javascript{9-21}
// web/src/components/Comment.test.js

import { render, screen } from '@redwoodjs/testing'

import Comment from './Comment'

describe('Comment', () => {
  it('renders successfully', () => {
    const comment = {
      name: 'John Doe',
      body: 'This is my comment',
      createdAt: '2020-01-02T12:34:56Z',
    }

    expect(screen.getByText(comment.name)).toBeInTheDocument()
    expect(screen.getByText(comment.body)).toBeInTheDocument()

    const dateExpect = screen.getByText('2 January 2020')
    expect(dateExpect).toBeInTheDocument()
    expect(dateElement.nodeName).toEqual('TIME')
    expect(dateExpect).toHaveAttribute('datetime', comment.createdAt)
  })
})

```

Here we're testing for both elements of the output `createdAt` timestamp: the actual text that's output (similar to how we tested for a blog post's truncated body) but also that the element that wraps that text is a `<time>` tag and that it contains a `datetime` attribute with the raw value of `comment.createdAt`. This might seem like overkill but the point of the `datetime` attribute is to provide a machine-readable timestamp that the browser could (theoretically) hook into and do stuff with. This makes sure that we preseve that ability!

> **What happens if we change the formatted output of the timestamp? Wouldn't we have to change the test?**
>
> Yes, just like we'd have to change the truncation text if we changed the length of the truncation. One alternative approach to testing for the formatted output could be to move the date formatting formula into a function that you can export from the Comment component. Then you can import that in your test and use it to check the formatted output. Now if you change the formula the test keeps passing because it's sharing the function with Comment.

## Multiple Comments

Let's think about where our comments are being displayed. Probably not on the homepage, since that only shows a summary of each post. A user would need to go to the full page to show the comments for that blog post. But that page is only fetching the data for the single blog post itself, nothing else. We'll need to get the comments and since we'll be fetching *and* displaying them, that sounds like a job for a Cell.

> **Couldn't the query for the blog post page also fetch the comments?**
>
> Yes, it could! But the idea behind Cells is to make components even more [composable](https://en.wikipedia.org/wiki/Composability) by having them be responsible for their own data fetching *and* display. If we rely on a blog post to fetch the comments then the new Comments component we're about to create now requires something else to fetch the comments and pass them in. If we re-use the Comments component somewhere, now we're fetching comments in two different places.
>
> **But what about the Comment component we just made, why doesn't that fetch its own data?**
>
> There aren't any instances I (the author) could think of where we would ever want to display only a single comment in isolation—it would always be a list of all comments on a post. If displaying a single Comment was common for your use case then it could definitely be converted to a CommentCell and have it responsible for pulling the data for that single comment itself. But keep in mind that if you have 50 comments on a blog post, that's now 50 GraphQL calls that need to go out, one for each comment. There's always a tradeoff!
>
> **Then why make a standalone Comment component at all? Why not just do all the display in the CommentsCell?**
>
> We're trying to start in small chunks to make the tutorial more digestable for a new audience so we're starting simple and getting more complex as we go. But it also just feels *nice* to build up a UI from these smaller chunks that are easier to reason about and keep separate in my (the author's) head.
>
> **But what about—**
>
> Look, we gotta end this sidebar and get back to building this thing. You can ask more questions later, promise!

### Storybook

Let's generate a `CommentsCell`:

```terminal
yarn rw g cell Comments
```

Storybook updates with a new **CommentsCell** under the **Cells** folder. Let's update the Success story to use the Comment component created earlier:

```javascript{3,20}
// web/src/components/CommentsCell/CommentsCell.js

import Comment from 'src/components/Comment'

export const QUERY = gql`
  query CommentsQuery {
    comments {
      id
    }
  }
`

export const Loading = () => <div>Loading...</div>

export const Empty = () => <div>Empty</div>

export const Failure = ({ error }) => <div>Error: {error.message}</div>

export const Success = ({ comments }) => {
  return comments.map((comment) => <Comment key={comment.id} comment={comment} />)
}
```

We're passing an additional `key` prop to make React happy when iterating over an array with `map`.

If you check Storybook, you'll seen an error. We'll need to update the `mock.js` file that came along for the ride when we generated the Cell so that it returns an array instead of just a simple object with some sample data:

```javascript{4-11}
// web/src/components/CommentsCell/CommentsCell.mock.js

export const standard = (/* vars, { ctx, req } */) => ({
  comments: [
    {
      id: 1, name: 'Rob Cameron', body: 'First comment', createdAt: '2020-01-02T12:34:56Z'
    },
    {
      id: 2, name: 'David Price', body: 'Second comment', createdAt: '2020-02-03T23:00:00Z'
    },
  ]
})

```

> What's this `standard` thing? Think of it as the standard, default mock if you don't do anything else. We would have loved to use the name "default" but that's already a reserved word in Javascript!

Storybook refreshes and we've got comments! We've got the same issue here where it's hard to see our rounded corners and also the two separate comments are are hard to distinguish because they're right next to each other:

![image](https://user-images.githubusercontent.com/300/95799544-dce60300-0ca9-11eb-9520-a1aac4ec46e6.png)

The gap between the two comments *is* a concern for this component, since it's responsible for drawing multiple comments and their layout. So let's fix that in CommentsCell:

```javascript
// web/src/components/CommentsCell/CommentsCell.js

export const Success = ({ comments }) => {
  return (
    <div className="-mt-8">
      {comments.map((comment) => (
        <div key={comment.id} className="mt-8">
          <Comment comment={comment} />
        </div>
      ))}
    </div>
  )
}
```

We had to move the `key` prop to the surrounding `<div>`. We then gave each comment a top margin and removed an equal top margin from the entire container to set it back to zero.

> Why a top margin and not a bottom margin? Remember when we said a component should be responsible for *it's own* display? If you add a bottom margin, that's one component influcing the one below it (which it shouldn't care about). Adding a *top* margin is this component moving *itself* down, which means it's again responsible for its own display.

Let's add a margin around the story itself, similar to what we did in the Comment story:

```javascript
// web/src/components/CommentsCell/CommentsCell.stories.js

export const success = () => {
  return Success ? (
    <div className="m-8 mt-16">
      <Success {...standard()} />
    </div>
  ) : null
}
```

> Why both `m-8` and `mt-16`? One of the fun rules of CSS is that if a parent and child both have margins, but no border or padding between them, their `margin-top` and `margin-bottom` [collapses](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_Box_Model/Mastering_margin_collapsing). So even though the story container will have a margin of 8 (which equals 2rem) remember that the container for CommentsCell has a -8 margin (-2rem). Those two collapse and essentially cancel each other out to 0 top margin. Setting `mt-16` sets a 4rem margin, which after subtracing 2rem leaves us with 2rem, which is what we wanted to start with!

![image](https://user-images.githubusercontent.com/300/95800481-4cf58880-0cac-11eb-9457-ff3f1f0d34b8.png)

Looking good! Let's add our CommentsCell to the actual blog post display page:

```javascript{4,21}
// web/src/components/BlogPost/BlogPost.js

import { Link, routes } from '@redwoodjs/router'
import CommentsCell from 'src/components/CommentsCell'

const truncate = (text, length) => {
  return text.substring(0, length) + '...'
}

const BlogPost = ({ post, summary = false }) => {
  return (
    <article className="mt-10">
      <header>
        <h2 className="text-xl text-blue-700 font-semibold">
          <Link to={routes.blogPost({ id: post.id })}>{post.title}</Link>
        </h2>
      </header>
      <div className="mt-2 text-gray-900 font-light">
        {summary ? truncate(post.body, 100) : post.body}
      </div>
      {!summary && <CommentsCell />}
    </article>
  )
}

export default BlogPost
```

If we are *not* showing the summary, then we'll show the comments. Take a look at the **Full** and **Summary** stories and you should see comments on one and not on the other.

Once again our component is bumping right up against the edges of the window. We've got two stories in this file and would have to manually add margins around both of them. Ugh. Luckily Storybook has a way to add styling to all stories using [decorators](https://storybook.js.org/docs/react/writing-stories/decorators). In the `default` export at the bottom of the story you can define a `decorators` key and the value is JSX that will wrap all the stories in the file automatically:

```javascript{5-7}
// web/src/components/BlogPost/BlogPost.js

export default {
  title: 'Components/BlogPost',
  decorators: [
    (Story) => <div className="m-8"><Story /></div>
  ]
}
```

Save, and both the **Full** and **Summary** stories should have margins around them now.

> For more extensive, global styling options, look into Storybook [theming](https://storybook.js.org/docs/react/configure/theming).

![image](https://user-images.githubusercontent.com/300/96509066-5d5bb500-1210-11eb-8ddd-8786b7033cac.png)

We could use a gap between the end of the blog post and the start of the comments to help separate the two:

```javascript{14-21}
// web/src/components/BlogPost/BlogPost.js

const BlogPost = ({ post, summary = false }) => {
  return (
    <article className="mt-10">
      <header>
        <h2 className="text-xl text-blue-700 font-semibold">
          <Link to={routes.blogPost({ id: post.id })}>{post.title}</Link>
        </h2>
      </header>
      <div className="mt-2 text-gray-900 font-light">
        {summary ? truncate(post.body, 100) : post.body}
      </div>
      {!summary && (
        <div className="mt-24">
         <CommentsCell />
        </div>
      )}
    </article>
  )
}

export default BlogPost
```

![image](https://user-images.githubusercontent.com/300/100682809-bfd5c400-332b-11eb-98e0-d2d526c1aa58.png)

Okay, comment display is looking good! However, you may have noticed that if you tried going to the actual site there's an error where the comments should be:

![image](https://user-images.githubusercontent.com/300/97620392-c0063b00-19de-11eb-84e8-d35f028d67b8.png)

Why is that? Remember that we started with the `CommentsCell`, but never actually created a Comment model in `schema.prisma` or created an SDL and service! That's another neat part of working with Storybook: you can build out UI functionality completely isolated from the api-side. In a team setting this is great because a web-side team can work on the UI while the api-side team can be building the backend end simultaneously and one doesn't have to wait for the other.

### Testing

We added one component (`<Comments>`) and edited another (`<BlogPost>`) so we'll want to add tests in both.

#### Testing Comments

The actual `<Comment>` component does most of the work so there's no need to test all of that functionality again. What things does `<CommentsCell`> do that make it unique?

* Has a loading message
* Has an error message
* Has a failure message
* When it renders succesfully, it outputs as many comments as were returned by the `QUERY`

The default `CommentsCell.test.js` actually tests every state for us, albeit at an absolute minimum level—it make sure no errors are thrown:

```javascript
import { render, screen } from '@redwoodjs/testing'
import { Loading, Empty, Failure, Success } from './CommentsCell'
import { standard } from './CommentsCell.mock'

describe('CommentsCell', () => {
  test('Loading renders successfully', () => {
    expect(() => {
      render(<Loading />)
    }).not.toThrow()
  })

  test('Empty renders successfully', async () => {
    expect(() => {
      render(<Empty />)
    }).not.toThrow()
  })

  test('Failure renders successfully', async () => {
    expect(() => {
      render(<Failure error={new Error('Oh no')} />)
    }).not.toThrow()
  })

  test('Success renders successfully', async () => {
    expect(() => {
      render(<Success comments={standard().comments} />)
    }).not.toThrow()
  })
})
```

And that's nothing to scoff at! As you've probably experienced, a React component usually either works 100% or throws an error. If it works, great! If it fails then the test fails too, which is exactly what we want to happen.

But in this case we can do a little more to make sure `<CommentsCell>` is doing what we expect. First let's update the `CommentsCell.mock.js` to send in two comments:

```javascript{5-16}
// web/src/components/CommentsCell/CommentsCell.mock.js

export const standard = (/* vars, { ctx, req } */) => ({
  comments: [
    {
      id: 1,
      name: 'Rob Cameron',
      body: 'First comment',
      createdAt: '2020-01-02T12:34:56Z',
    },
    {
      id: 2,
      name: 'David Price',
      body: 'Second comment',
      createdAt: '2020-02-03T23:00:00Z',
    },
  ],
})
```

And now let's update the `Success` test in `CommentsCell.test.js` to check that exactly the number of comments we passed in as a prop are rendered. How do we know a comment was rendered? How about if we check that each `comment.body` (the most important part of the comment) is present on the screen:

```javascript
// web/src/components/CommentsCell/CommentsCell.test.js

test('Success renders successfully', async () => {
  const comments = standard().comments
  render(<Success comments={comments} />)

  comments.forEach((comment) => {
    expect(screen.getByText(comment.body)).toBeInTheDocument()
  })
})
```

We're looping through each `comment` from the mock so that even if we add more later, we're covered.

#### Testing BlogPost

The functionality we added to `<BlogPost>` says to show the comments for the post if we are *not* showing the summary. We've got a test for both the "full" and "summary" renders already. Generally you want your tests to be testing "one thing" so let's add two additional tests for our new functionality:

```javascript{3,23-30,43-51}
// web/src/components/BlogPost/BlogPost.test.js

import { render, screen, waitFor } from '@redwoodjs/testing'

import BlogPost from './BlogPost'
import { standard } from 'src/components/CommentsCell/CommentsCell.mock'

const POST = {
  id: 1,
  title: 'First post',
  body: `Neutra tacos hot chicken prism raw denim, put a bird on it enamel pin post-ironic vape cred DIY. Street art next level umami squid. Hammock hexagon glossier 8-bit banjo. Neutra la croix mixtape echo park four loko semiotics kitsch forage chambray. Semiotics salvia selfies jianbing hella shaman. Letterpress helvetica vaporware cronut, shaman butcher YOLO poke fixie hoodie gentrify woke heirloom.`,
  createdAt: new Date().toISOString(),
}

describe('BlogPost', () => {
  it('renders a blog post', () => {
    render(<BlogPost post={POST} />)

    expect(screen.getByText(POST.title)).toBeInTheDocument()
    expect(screen.getByText(POST.body)).toBeInTheDocument()
  })

  it('renders comments when displaying a full blog post', async () => {
    const comment = standard().comments[0]
    render(<BlogPost post={POST} />)

    await waitFor(() =>
      expect(screen.getByText(comment.body)).toBeInTheDocument()
    )
  })

  it('renders a summary of a blog post', () => {
    render(<BlogPost post={POST} summary={true} />)

    expect(screen.getByText(POST.title)).toBeInTheDocument()
    expect(
      screen.getByText(
        'Neutra tacos hot chicken prism raw denim, put a bird on it enamel pin post-ironic vape cred DIY. Str...'
      )
    ).toBeInTheDocument()
  })

  it('does not render comments when displaying a summary', async () => {
    const comment = standard().comments[0]
    render(<BlogPost post={POST} summary={true} />)

    await waitFor(() =>
      expect(screen.queryByText(comment.body)).not.toBeInTheDocument()
    )
  })
})
```

We're introducting a new test function here `waitFor()` which will wait for things like GraphQL queries to finish running before checking for what's been rendered. Since `<BlogPost>` renders `<CommentsCell>` we need to wait for the `Success` component of `<CommentsCell>` to be rendered.

> The summary version of `<BlogPost>` does *not* render the `<CommentsCell>`, but we should still wait. Why? If we did mistakenly start including `<CommentsCell>`, but didn't wait for the render, we would get a falsely passing test—indeed the text isn't on the page but that's because it's still showing the `Loading` component! If we had waited we would have seen the actual comment body get rendered, and the test would (correctly) fail.

Okay we're finally ready to let users create their comments. But first we need to update the database to start storing them.

## Adding Comments to the Schema

Let's take a moment to appreciate how amazing this is—we built, designed and tested a completely new component for our app, which displays data from an API call (which would pull that data from a database) without actually having to build any of that backend functionality! Storybook and Jest let us provide fake data so we could get our component working.

Unfortunately, even with all of this flexibility there's still no such thing as a free lunch. Eventually we're going to have to actually do that backend work. Now's the time.

If you went through the first part of the tutorial you should be somewhat familiar with this flow:

1. Add a model to `schema.prisma`
2. Run a couple of `yarn rw db` commands to migrate the database
3. Generate an SDL and service

### Adding the Comment model

Let's do that now:

```javascript{17,29-36}
// api/prisma/schema.prisma

datasource DS {
  provider = "sqlite"
  url      = env("DATABASE_URL")
}

generator client {
  provider      = "prisma-client-js"
  binaryTargets = "native"
}

model Post {
  id        Int      @id @default(autoincrement())
  title     String
  body      String
  comments  Comment[]
  createdAt DateTime @default(now())
}

model Contact {
  id        Int      @id @default(autoincrement())
  name      String
  email     String
  message   String
  createdAt DateTime @default(now())
}

model Comment {
  id        Int      @id @default(autoincrement())
  name      String
  body      String
  post      Post     @relation(fields: [postId], references: [id])
  postId    Int
  createdAt DateTime @default(now())
}
```

Most of these lines look very similar to what we've already seen, but this is the first instance of a [relation](https://www.prisma.io/docs/reference/tools-and-interfaces/prisma-schema/relations) between two models. `Comment` gets two entries:

* `post` which has a type of `Post` and a special `@relation` keyword that tells Prisma how to connect a `Comment` to a `Post`. In this case the field `postId` references the field `id` in `Post`
* `postId` is just a regular `Int` column which contains the `id` of the `Post` that this comment is referencing

This gives us a classic database model:

```
┌───────────┐       ┌───────────┐
│   Post    │       │  Comment  │
├───────────┤       ├───────────┤
│ id        │───┐   │ id        │
│ title     │   │   │ name      │
│ body      │   │   │ body      │
│ createdAt │   └──<│ postId    │
└───────────┘       │ createdAt │
                    └───────────┘
```

Note that there is no column called `post` on `Comment`—this is special syntax for Prisma to know how to connect the models together and for you to reference that connection. When you query for a `Comment` using Prisma you can get access to the attached `Post` using that name:

```javascript
db.comment.findOne({ where: { id: 1 }}).post()
```

We also added a convenience `comments` field to `Post` which gives us the same capability in reverse:

```javascript
db.post.findOne({ where: { id: 1 }}).comments()
```

### Running the Migration

This one is easy enough: we'll create a new migration with a name and then run it:

```terminal
yarn rw db save create comments
yarn rw db up
```

### Creating the SDL and Service

Next we'll create the SDL (that defines the GraphQL interface) and a service (to get the records out of the database):

```terminal
yarn rw g sdl comment
```

That command will create both the SDL and the service. And if you take a look back in the browser you should see a different message than the error we were seeing before:

![image](https://user-images.githubusercontent.com/300/97620271-8fbe9c80-19de-11eb-8d95-86c1cb7bcb91.png)

"Empty" means the Cell rendered correctly! There just aren't any comments in the database yet. Let's update the `<CommentsCell>` component to make that "Empty" message a little more friendly:

```javascript
// web/src/components/CommentsCell/CommentsCell.js

export const Empty = () => {
  return <div className="text-center text-gray-500">No comments yet</div>
}
```

That's better. Let's update the test that covers the Empty component render as well:

```javascript
// web/src/components/CommentsCell/CommentsCell.test.js

test('Empty renders a "no comments" message', () => {
  render(<Empty />)
  expect(screen.getByText('No comments yet')).toBeInTheDocument()
})
```

Okay, let's focus on the service for bit. We'll need to add a function to let users create a new comment and we'll add a test that covers the new functionality.

### Building out the Service

By virtue of using the generator we've already got the function we need to select all comments from the database:

```javascript
// api/src/services/comments/comments.js

export const comments = () => {
  return db.comment.findMany()
}
```

> Have you noticed that something may be amiss? This function returns *all* comments, and all comments only. Could this come back to bite us?
>
> To be continued...

We need to be able to create a comment as well. We'll use the same convention we use in Redwood's generated scaffolds: the create endpoint will accept a single parameter `input` which is an object with the individual model fields:

```javascript
// api/src/services/comments/comments.js

export const createComment = ({ input }) => {
  return db.comment.create({
    data: input,
  })
}
```

We'll also need to expose this function via GraphQL so we'll add a Mutation to the SDL:

```graphql
// api/src/graphql/comments.sdl.js

type Mutation {
  createComment(input: CreateCommentInput!): Comment!
}
```

> The `CreateCommentInput` type was already created for us by the SDL generator.

That's all we need to create a comment! But let's think for a moment: is there anything else we need to do with a comment? Let's make the decision that users won't be able to update an existing comment. And we don't need to select individual comments (remember earlier we talked about the possibility of each comment being responsible for its own API request and display, but we decided against it).

What about deleting a comment? We won't let a user delete their own comment, but as owners of the blog we should be able to delete/moderate them. So we'll need a delete function and API endpoint as well. Let's add those:

```javascript
// api/src/services/comments/comments.js

export const deleteComment = ({ id }) => {
  return db.comment.delete({
    where: { id },
  })
}
```

```graphql{5}
// api/src/graphql/comments.sdl.js

type Mutation {
  createComment(input: CreateCommentInput!): Comment!
  deleteComment(id: Int!): Comment!
}
```

### Testing the Service

Let's make sure our service functionality is working and continues to work as we modify our app.

If you open up `api/src/services/comments/comments.test.js` you'll see there's one in there already, making sure that retrieving all comments (the default `comments()` function that was generated along with the service) works:

```javascript
// api/src/services/comments/comments.test.js

import { comments } from './comments'

describe('comments', () => {
  scenario('returns a list of comments', async (scenario) => {
    const list = await comments()

    expect(list.length).toEqual(Object.keys(scenario.comment).length)
  })
})
```

What is this `scenario()` function? That's made available by Redwood that mostly acts like Jest's built-in `it()` and `test()` functions, but with one important difference: it pre-seeds a test database with data that is then passed to you in the `scenario` argument. You can count on this data existing in the database and being reset between tests in case you make changes to it.

Where does that data come from? Take a look at the `comments.scenarios.js` file which is next door:

```javascript
export const standard = scenario({
  comment: {
    one: {
      name: 'String',
      body: 'String',
      post: { create: { title: 'String', body: 'String' } },
    },

    two: {
      name: 'String',
      body: 'String',
      post: { create: { title: 'String', body: 'String' } },
    },
  },
})
```

This also calls a `scenario()` function, but this one assures that your data structure matches what's defined in Prisma. It just returns the same object you give it.

> **The "standard" scenario**
>
> The exported scenario here is named "standard." Remember when we worked on component tests and mocks, there was a special mock named `standard` which Redwood would use by default if you didn't specify a name? The same rule applies here! When we add a test for `createComment()` we'll see an example of using a different scenario with a unique name.

The nested structure of a scenario is defined like this:

* **comment**: the name of the model this data is for
  * **one, two**: a friendly name given to the scenario data which you can reference in your tests
    * **name, message, post**: the actual data that will be put in the database. In this case a **Comment** requires that it be related to a **Post**, so the scenario has a `post` key and values as well (using Prisma's [nested create syntax](https://www.prisma.io/docs/concepts/components/prisma-client/relation-queries#nested-writes))

When you receive the `scenario` argument in your test you can follow the same object nesting in order to reference the fields, like `scenario.comment.one.name`.

> **Why is every field just containing the string "String"?**
>
> When generating the service (and the test and scenarios) all we (Redwood) knows about your data is the types for each field as defined in `schema.prisma`, namely `String`, `Integer` or `DateTime`. So we add the simplest data possible that fulfills the type requirement by Prisma to get the data into the database. You should definitely replace this data with something that looks more like the real data your app will be expecting. In fact...

Let's replace that scenario data with something more like what we expect to see in our app:

```javascript
// api/src/services/comments/comments.scenarios.js

export const standard = scenario({
  comment: {
    jane: {
      name: 'Jane Doe',
      message: 'I like trees',
      post: {
        create: {
          title: 'Redwood Leaves',
          body: 'The quick brown fox jumped over the lazy dog.'
        }
      }
    },
    john: {
      name: 'John Doe',
      message: 'Hug a tree today',
      post: {
        create: {
          title: 'Root Systems',
          body: 'The five boxing wizards jump quickly.'
        }
      }
    }
  }
})
```

The test created by the service generator simply checks to make sure the same number of records are returned so changing the content of the data here won't affect the test.

#### Testing createComment()

Let's add our first service test by making sure that `createComment()` actually stores a new comment in the database. When creating a comment we're not as worried about existing data in the database so let's create a new scenario which only contains a post—the post we'll be linking the new comment to through the comment's `postId` field:

```javascript
// api/src/services/comments/comments.scenarios.js

export const postOnly = scenario({
  post: {
    bark: {
      title: 'Bark',
      body: 'Sphinx of black quartz, judge my vow.'
    }
  }
})
```

Now we can pass the `postOnly` scenario name as the first argument to a new `scenario()` test:

```javascript{3,12-25}
// api/src/services/comments/comments.test.js

import { comments, createComment } from './comments'

describe('comments', () => {
  scenario('returns a list of comments', async (scenario) => {
    const list = await comments()

    expect(list.length).toEqual(Object.keys(scenario.comment).length)
  })

  scenario('postOnly', 'creates a new comment', async (scenario) => {
    const comment = await createComment({
      input: {
        name: 'Billy Bob',
        body: "A tree's bark is worse than its bite",
        postId: scenario.post.bark.id
      }
    })

    expect(comment.name).toEqual('Billy Bob')
    expect(comment.body).toEqual("A tree's bark is worse than its bite")
    expect(comment.postId).toEqual(scenario.post.park.id)
    expect(comment.createdAt).not.toEqual(null)
  })
})
```

We pass an optional first argument to `scenario()` which is the named scenario to use, instead of the default of "standard."

We were able to use the `id` of the post that we created in our scenario because the scenarios contain the actual database data after being inserted, not just the few fields we defined in the scenario itself. In addition to `id` we could access `createdAt` which is defaulted to `now()` in the database.

We'll test that all the fields we give to the `createComment()` function are actually created in the database, and for good measure just make sure that `createdAt` is set to a non-null value. We could test that the actual timestamp is correct, but that involves freezing the Javascript Date object so that no matter how long the test takes, you can still compare the value to `new Date` which is right *now*, down to the millisecond. While possible, it's beyond the scope of our easy, breezy tutorial since it gets [very gnarly](https://codewithhugo.com/mocking-the-current-date-in-jest-tests/)!

Okay, our comments service is feeling pretty solid now that we have our tests in place. The last step is add a form so that users can actually leave a comment on a blog post.

## Creating a Comment Form

Let's generate a form and then we'll build it out and integrate it via Storybook, then add some tests.

```terminal
yarn rw g component CommentForm
```

And startup Storybook again if it isn't still running:

```terminal
yarn rw storybook
```

You'll see that there's a **CommentForm** entry in Storybook now, ready for us to get started.

### Storybook

Let's build a simple form to take the user's name and their comment and add some styling to match it to the blog:

```javascript
// web/src/components/CommentForm/CommentForm.js

import { Form, FormError, Label, TextField, TextAreaField, Submit } from '@redwoodjs/forms'

const CommentForm = () => {
  return (
    <div>
      <h3 className="font-light text-lg text-gray-600">Leave a Comment</h3>
      <Form className="mt-4 w-full">
        <FormError
          error={error}
          titleClassName="font-semibold"
          wrapperClassName="bg-red-100 text-red-900 text-sm p-3 rounded"
        />
        <Label name="name" className="block text-sm text-gray-600 uppercase">
          Name
        </Label>
        <TextField
          name="name"
          className="block w-full p-1 border rounded text-xs "
          validation={{ required: true }}
        />

        <Label
          name="body"
          className="block mt-4 text-sm text-gray-600 uppercase"
        >
          Comment
        </Label>
        <TextAreaField
          name="body"
          className="block w-full p-1 border rounded h-24 text-xs"
          validation={{ required: true }}
        />

        <Submit
          className="block mt-4 bg-blue-500 text-white text-xs font-semibold uppercase tracking-wide rounded px-3 py-2 disabled:opacity-50"
        >
          Submit
        </Submit>
      </Form>
    </div>
  )
}

export default CommentForm
```

Note that the form and its inputs are set to 100% width. Again, the form shouldn't be dictating anything about its layout that its parent should be resposible for, like how wide the inputs are. Those should be determined by whatever contains it so that it looks good with the rest of the content on the page. So the form will be 100% wide and the parent (whoever that ends up being) will decide how wide it really is on the page.

And let's add some margin around the whole component in Storybook so that the 100% width doesn't run into the Storybook frame:

```javascript{7,9}
// web/src/components/CommentForm/CommentForm.stories.js

import CommentForm from './CommentForm'

export const generated = () => {
  return (
    <div className="m-4">
      <CommentForm />
    </div>
  )
}

export default { title: 'Components/CommentForm' }
```

![image](https://user-images.githubusercontent.com/300/100663134-b5ef9900-330a-11eb-8ba3-e9e4bfe89b84.png)

You can even try submitting the form right in Storybook! If you leave "name" or "comment" blank then they should get focus when you try to submit, indicating that they are required. If you fill them both in and click **Submit** nothing happens because we haven't hooked up the submit yet. Let's do that now.

### Submitting

Submitting the form should use the `createComment` function we added to our services and GraphQL. We'll need to add a mutation to the form component and an `onSubmit` hander to the form so that the create can be called with the data in the form:

```javascript{4,6-15,18-22,58}
// web/src/components/CommentForm/CommentForm.js

import { Form, FormError, Label, TextField, TextAreaField, Submit } from '@redwoodjs/forms'
import { useMutation } from '@redwoodjs/web'

const CREATE = gql`
  mutation CreateCommentMutation($input: CreateCommentInput!) {
    createComment(input: $input) {
      id
      name
      body
      createdAt
    }
  }
`

const CommentForm = () => {
  const [createComment, { loading }] = useMutation(CREATE)

  const onSubmit = (input) => {
    createComment({ variables: { input } })
  }

  return (
    <div>
      <h3 className="font-light text-lg text-gray-600">Leave a Comment</h3>
      <Form className="mt-4 w-full" onSubmit={onSubmit}>
        <FormError
          error={error}
          titleClassName="font-semibold"
          wrapperClassName="bg-red-100 text-red-900 text-sm p-3 rounded"
        />
        <Label
          name="name"
          className="block text-xs font-semibold text-gray-500 uppercase"
        >
          Name
        </Label>
        <TextField
          name="name"
          className="block w-full p-1 border rounded text-sm "
          validation={{ required: true }}
        />

        <Label
          name="body"
          className="block mt-4 text-xs font-semibold text-gray-500 uppercase"
        >
          Comment
        </Label>
        <TextAreaField
          name="body"
          className="block w-full p-1 border rounded h-24 text-sm"
          validation={{ required: true }}
        />

        <Submit
          disabled={loading}
          className="block mt-4 bg-blue-500 text-white text-xs font-semibold uppercase tracking-wide rounded px-3 py-2 disabled:opacity-50"
        >
          Submit
        </Submit>
      </Form>
    </div>
  )
}

export default CommentForm
```

If you try to submit the form you'll get an error in the web console—Storybook doesn't actually start up a real GraphQL server to handle the request! But we can mock the request in the story and handle the response ourselves:

```javascript{6-18}
// web/src/components/CommentForm/CommentForm.stories.js

import CommentForm from './CommentForm'

export const generated = () => {
  mockGraphQLMutation('CreateCommentMutation', (variables, { ctx }) => {
    const id = parseInt(Math.random() * 1000)
    ctx.delay(1000)

    return {
      comment: {
        id,
        name: variables.input.name,
        body: variables.input.body,
        createdAt: new Date().toISOString(),
      },
    }
  })

  return (
    <div className="m-4">
      <CommentForm />
    </div>
  )
}

export default { title: 'Components/CommentForm' }
```

To use `mockGraphQLMutation` you call it with the name of the mutation you want to intercept and then the function that will handle the interception and return a response. The arguments passed to that function give us some flexibility in how we handle the response.

In our case we want the `variables` that were passed to the mutation (the `name` and `body`) as well as the context object (abbreviated as `ctx`) so that we can add a delay to simulate a round trip to the server. This will let us test that the **Submit** button is disabled for that one second and you can't submit a second comment while the first one is still being saved.

Try out the form now and the error should be gone. Also the **Submit** button should become visually disabled and clicking it during that one second delay does nothing.

Once our comment is sent notice that the comment form still contains their entries. That's not ideal. Let's clear that out. We did something [similar](/tutorial/saving-data#one-more-thing) back in Part 1 of the tutorial when we created the Contact form:

```javascript{5,19-22,33}
// web/src/components/CommentForm/CommentForm.js

import { Form, FormError, Label, TextField, TextAreaField, Submit } from '@redwoodjs/forms'
import { useMutation } from '@redwoodjs/web'
import { useForm } from 'react-hook-form'

const CREATE = gql`
  mutation CreateCommentMutation($input: CreateCommentInput!) {
    createComment(input: $input) {
      id
      name
      body
      createdAt
    }
  }
`

const CommentForm = () => {
  const formMethods = useForm()
  const [createComment, { loading }] = useMutation(CREATE, {
    onCompleted: formMethods.reset,
  })

  const onSubmit = (input) => {
    createComment({ variables: { input } })
  }

  return (
    <div>
      <h3 className="font-light text-lg text-gray-600">Leave a Comment</h3>
      <Form
        className="mt-4 w-full"
        formMethods={formMethods}
        onSubmit={onSubmit}
      >
        <FormError
          error={error}
          titleClassName="font-semibold"
          wrapperClassName="bg-red-100 text-red-900 text-sm p-3 rounded"
        />
        <Label
          name="name"
          className="block text-xs font-semibold text-gray-500 uppercase"
        >
          Name
        </Label>
        <TextField
          name="name"
          className="block w-full p-1 border rounded text-sm "
          validation={{ required: true }}
        />

        <Label
          name="body"
          className="block mt-4 text-xs font-semibold text-gray-500 uppercase"
        >
          Comment
        </Label>
        <TextAreaField
          name="body"
          className="block w-full p-1 border rounded h-24 text-sm"
          validation={{ required: true }}
        />

        <Submit
          disabled={loading}
          className="block mt-4 bg-blue-500 text-white text-xs font-semibold uppercase tracking-wide rounded px-3 py-2 disabled:opacity-50"
        >
          Submit
        </Submit>
      </Form>
    </div>
  )
}

export default CommentForm
```

We extract the `formMethods` so that we can call `reset()` ourselves. We do that once the mutation completes.

### Adding the Form to the Blog Post

Right above the display of existing comments on a blog post is probably where our form should go. So should we add it to the **BlogPostCell** along with the **CommentsCell** component? If wherever we display a list of comments we'll also include the form to add a new one, that feels like it may as well just go into the **CommentsCell** component itself. Let's put it there:

```javascript{4,28}
// web/components/CommentsCell/CommentsCell.js

import Comment from 'src/components/Comment'
import CommentForm from 'src/components/CommentForm'

export const QUERY = gql`
  query CommentsQuery {
    comments {
      id
      name
      body
      createdAt
      postId
    }
  }
`

export const Loading = () => <div>Loading...</div>

export const Empty = () => {
  return <div className="text-center text-gray-500">No comments yet</div>
}

export const Failure = ({ error }) => <div>Error: {error.message}</div>

export const Success = ({ comments }) => {
  return (
    <div className="-mt-8">
      <CommentForm />
      {comments.map((comment, i) => (
        <div key={i} className="mt-8">
          <Comment comment={comment} />
        </div>
      ))}
    </div>
  )
}
```

Thanks to our planning ahead and having each component only responsible for its own layout, everything automatically looks great! Here it is in the **Success** component of the **CommentsCell**:

![image](https://user-images.githubusercontent.com/300/100682327-b39d3700-332a-11eb-80a0-76be6217455c.png)

And the **BlogPost** component:

![image](https://user-images.githubusercontent.com/300/100776202-19d09b00-33b9-11eb-99b5-05006fb231b6.png)

Awesome! How about we go over to the actual dev server now and try it out?

![image](https://user-images.githubusercontent.com/300/100779202-e0019380-33bc-11eb-8821-1833b13c34e9.png)

What the? Where's our form?

Remember how cells render themselves: if there are no records returned from the GraphQL call then the Cell will render the **Empty** component. In this case we only put the **CommentForm** into the **Success** component. So it's not showing the form at all!

> How come it worked fine in Storybook? Remember that Storybook is using fake data from our `*.mock.js` files, but the dev server is showing data from the actual database. There aren't any comments in the database yet!

Time for a decision:

1. Copy the **CommentForm** to the **Empty** component so that it still renders even when there are no comments in the database yet
2. Move the **CommentForm** somewhere else, maybe into **BlogPost** itself, around the same place that the **CommentsCell** is included.

You'll have to consider your tolerance for duplication and your adherance to the separation of concerns when deciding where it should go. For the purposes of the tutorial we're just going to go with option #2 and move the **CommentForm** component over to **BlogPost** instead (remember to remove it from **CommentsCell** as well):

```javascript{5,23-24,28}
// web/src/components/BlogPost/BlogPost.js

import { Link, routes } from '@redwoodjs/router'
import CommentsCell from 'src/components/CommentsCell'
import CommentForm from 'src/components/CommentForm'

const truncate = (text, length) => {
  return text.substring(0, length) + '...'
}

const BlogPost = ({ post, summary = false }) => {
  return (
    <article>
      <header>
        <h2 className="text-xl text-blue-700 font-semibold">
          <Link to={routes.blogPost({ id: post.id })}>{post.title}</Link>
        </h2>
      </header>
      <div className="mt-2 text-gray-900 font-light">
        {summary ? truncate(post.body, 100) : post.body}
      </div>
      {!summary && (
        <div className="mt-16">
          <CommentForm />
          <div className="mt-24">
            <CommentsCell />
          </div>
        </div>
      )}
    </article>
  )
}

export default BlogPost
```

That looks better!

![image](https://user-images.githubusercontent.com/300/100779113-c06a6b00-33bc-11eb-9112-0f7fc30a3f22.png)

Now comes the ultimate test: creating a comment! LET'S DO IT:

![image](https://user-images.githubusercontent.com/300/100806468-5d40fe80-33e5-11eb-89f7-e4b504078eff.png)

When we created our data schema we said that a post belongs to a comment via the `postId` field. And that field is required, so the GraphQL server is rejecting the request because we're not including that field. We're only sending `name` and `body`. Luckily we have access to the ID of the post we're commenting on thanks to the `post` object that's being passed into **BlogPost** itself!

> **Why didn't the story we wrote earlier expose this problem?**
>
> We manually mocked the GraphQL response in the story, and our mock always returns a correct response, regardless of the input!
>
> There's always a tradeoff when creating mock data—it greatly simplifies testing by not having to rely on the entire GraphQL stack, but that means if you want it to be as accurate as the real thing you basically need to *re-write the real thing in your mock*. In this case, leaving out the `postId` was a one-time fix so it's probably not worth going through the work of creating a story/mock/test that simulates what would happen if we left it off.
>
> But, if **CommentForm** ended up being a component that was re-used throughout your application, or the code itself will go through a lot of churn because other developers will constantly be making changes to it, it might be worth investing the time to make sure the interface (the props passed to it and the expected return) are exactly what you want them to be.

First let's pass the post's ID as a prop to **CommentForm**:

```javascript{16}
// web/src/components/BlogPost/BlogPost.js

const BlogPost = ({ post, summary = false }) => {
  return (
    <article>
      <header>
        <h2 className="text-xl text-blue-700 font-semibold">
          <Link to={routes.blogPost({ id: post.id })}>{post.title}</Link>
        </h2>
      </header>
      <div className="mt-2 text-gray-900 font-light">
        {summary ? truncate(post.body, 100) : post.body}
      </div>
      {!summary && (
        <div className="mt-16">
          <CommentForm postId={post.id} />
          <div className="mt-24">
            <CommentsCell />
          </div>
        </div>
      )}
    </article>
  )
}
```

And then we'll append that ID to the `input` object that's being passed to `createComment` in the **CommentForm**:

```javascript{3,10}
// web/src/components/CommentForm/CommentForm.js

const CommentForm = ({ postId }) => {
  const formMethods = useForm()
  const [createComment, { loading, error }] = useMutation(CREATE, {
    onCompleted: formMethods.reset,
  })

  const onSubmit = (input) => {
    createComment({ variables: { input: { postId, ...input } } })
  }

  return (
    //...
  )
}
```

Now fill out the comment form and submit! And...nothing happened! Believe it or not that's actually an improvement in the situation—no more error! What if we reload the page?

![image](https://user-images.githubusercontent.com/300/100950150-98fcc680-34c0-11eb-8808-944637b5ca1f.png)

Yay! It would have been nicer if that comment appeared as soon as we submitted the comment, so maybe that's a half-yay? But, we can fix that! It involves telling the GraphQL client (Apollo) that we created a new record and, if it would be so kind, to try the query again that gets the comments for this page.

### GraphQL Query Caching

Much has been written about the [complexities](https://medium.com/swlh/how-i-met-apollo-cache-ee804e6485e9) of [Apollo](https://medium.com/@galen.corey/understanding-apollo-fetch-policies-705b5ad71980) [caching](https://levelup.gitconnected.com/basics-of-caching-data-in-graphql-7ce9489dac15), but for the sake of brevity (and sanity) we're going to do the easiest thing that works, and that's tell Apollo to just re-fetch the query that shows comments in the cell.

Along with the variables you pass to a mutation function (`createComment` in our case) there's another option named `refetchQueries` where you pass an array of queries that should be re-run because, presumably, the data you just mutated is reflected in the result of those queries. In our case there's a single query, the **QUERY** export of **CommentsCell**. We'll import that at the top of **CommentForm** (and rename so it's clear what it is to the rest of our code) and then pass it along to the `refetchQueries` option:

```javascript{13,21}
// web/src/components/CommentForm/CommentForm.js

import {
  Form,
  FormError,
  Label,
  TextField,
  TextAreaField,
  Submit,
} from '@redwoodjs/forms'
import { useMutation } from '@redwoodjs/web'
import { useForm } from 'react-hook-form'
import { QUERY as CommentsQuery } from 'src/components/CommentsCell'

// ...

const CommentForm = ({ postId }) => {
  const formMethods = useForm()
  const [createComment, { loading, error }] = useMutation(CREATE, {
    onCompleted: formMethods.reset,
    refetchQueries: [{ query: CommentsQuery }],
  })

  //...
}
```

Now when we create a comment it appears right away! It might be hard to tell because it's at the bottom of the comments list (which is a fine position if you want to read comments in chronological order, oldest to newest). Let's pop up a little notification that the comment was successful to let the user know their contribution was successful.

We'll make use of good old fashioned React state to keep track of whether a comment has been posted in the form yet or not. If so, let's remove the comment form completely and show a "Thanks for your comment" message. We'll remove the form and show the message with just a couple of CSS classes:

```javascript{14,20,23-26,35,37-43,45}
// web/src/components/CommentForm/CommentForm.js

import {
  Form,
  FormError,
  Label,
  TextField,
  TextAreaField,
  Submit,
} from '@redwoodjs/forms'
import { useMutation } from '@redwoodjs/web'
import { useForm } from 'react-hook-form'
import { QUERY as CommentsQuery } from 'src/components/CommentsCell'
import { useState } from 'react'

// ...

const CommentForm = ({ postId }) => {
  const formMethods = useForm()
  const [hasPosted, setHasPosted] = useState(false)
  const [createComment, { loading, error }] = useMutation(CREATE, {
    onCompleted: () => {
      formMethods.reset()
      setHasPosted(true)
    },
    refetchQueries: [{ query: CommentsQuery }],
  })

  const onSubmit = (input) => {
    createComment({ variables: { input: { postId, ...input } } })
  }

  return (
    <div className="relative">
      <h3 className="font-light text-lg text-gray-600">Leave a Comment</h3>
      <div
        className={`${
          hasPosted ? 'absolute' : 'hidden'
        } flex items-center justify-center w-full h-full text-lg`}
      >
        <h4 className="text-green-500">Thank you for your comment!</h4>
      </div>
      <Form
        className={`mt-4 w-full ${hasPosted ? 'invisible' : ''}`}
        formMethods={formMethods}
        onSubmit={onSubmit}
        error={error}
      >
      //...
```

![image](https://user-images.githubusercontent.com/300/100949950-2d1a5e00-34c0-11eb-8c1c-3c9f925c6ecb.png)

We used `invisible` to just hide the form but have it still take up as much vertical space as it did before so that the comments don't suddenly jump up the page, which could be a little jarring.

So it looks like we're just about done here! Try going back to the homepage and go to another blog post. Let's bask in the glory of our amazing coding abilities and—OH NO:

![image](https://user-images.githubusercontent.com/300/100950583-7d45f000-34c1-11eb-8975-2c6f22c67843.png)

Every post has the same comments! **WHAT HAVE WE DONE??**

Remember our foreshadowing callout a few pages back, wondering if our `comments()` service which only returns *all* comments could come back to bite us? It finally has: when we get the comments for a post we're not actually getting them for only that post. We're ignoring the `postId` completely and just returning *all* comments! Turns out the old axiom is true: computers only do exactly what you tell them to. :(

Let's fix it!

### Returning Only Some Comments

Just like Redwood is split into a web- and api-side, we'll need to make both frontend and backend changes to get only some comments to show. Let's start with the backend and do a little test-driven development to make this change.

#### Updating the Service

Try running the test suite (or if it's already running take a peek at that terminal window) and make sure all of our tests still pass. The "lowest level" of the api-side is the services, so let's start there. Open up the **comments** service test and let's update it expect a new `postId` argument to be passed to the `comments()` function, the contents of which will be the `postId` of one of the comments that are created in our scenario:

```javascript{4}
// api/src/services/comments/comments.test.js

scenario('returns all comments from the database', async (scenario) => {
  const result = await comments({ postId: scenario.comment.one.postId })

  expect(result.length).toEqual(Object.keys(scenario.comment).length)
})
```

When the test suite runs everything will still pass. Javascript won't care if you're passing an argument all of a sudden (although if you were using Typescript you will actually get an error at this point!). In TDD you generally want to get your test to fail before adding code to the thing you're testing which will then cause the test to pass. What's something in this test that will be different once we're only returning *some* comments? How about the number of comments expected to be returned?

Based on our current scenario, each comment will also get associated with its own, unique post. So of the two comments in our scenario, only one should be returned for a given `postId`:

```javascript{6}
// api/src/services/comments/comments.test.js

scenario('returns all comments from the database', async (scenario) => {
  const result = await comments({ postId: scenario.comment.one.postId })

  expect(result.length).toEqual(1)
})
```

Now it should fail! Before we get it passing again, let's also change the name of the test to reflect what it's actually testing:

```javascript{3}
// api/src/services/comments/comments.test.js

scenario('returns all comments for a single post from the database', async (scenario) => {
  const result = await comments({ postId: scenario.comment.one.postId })

  expect(result.length).toEqual(1)
})
```

Okay, open up the actual `comments.js` service and we'll update it to accept the `postId` arugment and use it as an option to `findMany()`:

```javascript{3,4}
// api/src/services/comments/comments.js

export const comments = ({ postId }) => {
  return db.comment.findMany({ where: { postId } })
}
```

Save that and the test should pass again!

#### Updating GraphQL

Next we need to let GraphQL know that it should expect a `postId` to be passed for the `comments` query, and it's required (we don't currently have any view that allows you see all comments everywhere so we can ask that it always be present for now):

```javascript{4}
// api/src/graphql/comments.sdl.js

type Query {
  comments(postId: Int!): [Comment!]!
}
```

Now if you try refreshing the real site in dev mode you'll see an error where the comments should be displayed:

![image](https://user-images.githubusercontent.com/300/100953652-de70c200-34c7-11eb-90a5-55ca5d61d657.png)

If you inspect that error in the web inspector you'll see that it's complaining about `postId` not being present—exactly what we want!

That completes the backend updates, now we just need to tell **CommentsCell** to pass through the `postId` to the GraphQL query it makes.

#### Updating the Cell

First we'll need to get the `postId` to the cell itself. Remember when we added a `postId` prop to the **CommentForm** component so it knew which post to attach the new comment to? Let's do the same for **CommentsCell**. Open up **BlogPost**:

```javascript{18}
// web/src/components/BlogPost/BlogPost.js

const BlogPost = ({ post, summary = false }) => {
  return (
    <article>
      <header>
        <h2 className="text-xl text-blue-700 font-semibold">
          <Link to={routes.blogPost({ id: post.id })}>{post.title}</Link>
        </h2>
      </header>
      <div className="mt-2 text-gray-900 font-light">
        {summary ? truncate(post.body, 100) : post.body}
      </div>
      {!summary && (
        <div className="mt-16">
          <CommentForm postId={post.id} />
          <div className="mt-24">
            <CommentsCell postId={post.id} />
          </div>
        </div>
      )}
    </article>
  )
}
```

And finally, we need to take that `postId` and pass it on to the `QUERY` in the cell:

```javascript{4,5}
// web/src/components/CommentsCell/CommentsCell.js

export const QUERY = gql`
  query CommentsQuery($postId: Int!) {
    comments(postId: $postId) {
      id
      name
      body
      createdAt
      postId
    }
  }
`
```

Where does this magical `$postId` come from? Redwood is nice enough to automatically provide it to you since you passed it in as a prop when you called the component!

Try going to a couple of different blog posts and now only the first should show comment(s). You can add a comment to each blog post individually and they'll stick to their proper owners:

![image](https://user-images.githubusercontent.com/300/100954162-de24f680-34c8-11eb-817b-0a7ad802f28b.png)

However, you may have noticed that now when you post a comment it no longer appears right away! ARGH! Okay, turns out there's one more thing we need to do. Remember when we told the comment creation logic to `refetchQueries`? We need to include any variables that were present the first time so that it can refetch the proper ones.

#### Updating the Form

Okay this is the last fix, promise!

```javascript{8}
// web/src/components/CommentForm/CommentForm.js

const [createComment, { loading, error }] = useMutation(CREATE, {
  onCompleted: () => {
    formMethods.reset()
    setHasPosted(true)
  },
  refetchQueries: [{ query: CommentsQuery, variables: { postId } }],
})
```

There we go, comment engine complete! Our blog is totally perfect and there's absolutely nothing we could do to make it better.

Or is there?

## Role-Based Authorization Control (RBAC)

Imagine a few weeks in the future of our blog when every post hits the front page of the New York Times and we're getting hundreds of comments a day. We can't be expected to come up with quality content each day *and* moderate the endless stream of (mostly well-meaning) comments! We're going to need help. Let's hire a comment moderator to remove obvious spam and bad intentioned posts and help make the internet a better place.

We already have a login system for our blog, but right now it's all-or-nothing: you either get access to create blog posts, or you don't. In this case our comment moderator(s) will need logins so that we know who they are, but we're not going let them create new blog posts. We need some of role that we can give to our two kinds of users so we can distinguish them from one another.

Enter role-based authorization control, thankfully shortened to the common phrase **RBAC**. Authentication says who the person, authorization says what they can do. Currently the blog has the lowest common denominator of authroization: if they are logged in, they can do everything. Let's add a "less than anything, but more than nothing" level.

### Definining Roles

If you remember back in the first part of the tutorial we actually [pointed out](/tutorial/authentication#authentication-generation) that Netlify Identity provides an optional array of roles that you can attach to a user. That's exactly what we need!

> **What about other auth providers besides Netlify?**
>
> Some auth providers have a similar data structure that you can attach to a user, but if not you'll need to rely on your own database. Read more in the [RBAC Cookbook](/cookbook/role-based-access-control-rbac.html#roles-from-a-database).

If you started with your own blog code from Part 1 of the tutorial and already have it deployed on Netlify, you're ready to continue! If you cloned the [redwood-tutorial](https://github.com/redwoodjs/redwood-tutorial) code from GitHub you'll need to [create a Netlify site and deploy it](/tutorial/deployment), then [enable Netlify Identity](/tutorial/authentication#netlify-identity-setup) as described in the first part of the tutorial.

First we'll want to create a new user that will represent the comment moderator. You can use a completely different email address (if you have one), but if not you can use **The Plus Trick** to create a new, unique email address as far as Netlify is concerned, but that is actually the same as your original email address! **Note that not all email providers support this syntax, but the big ones like Gmail do.**

> The Plus Trick is a very handy feature of the email standard known as a "boxname", the idea being that you may have other inboxes besides one just named "Inbox" and by adding +something to your email address you can specify which inbox the mail should be sorted into. They don't appear t be in common usage these days, but they are ridiculously helpful for us developers when we're constantly needing new email addresses for testing!
>
> Use append +something to your email address before the @:
>
> * jane.doe+testing@example.com
> * john-doe+sample@example.com

Add your user and then edit them, adding a role of "moderator" in the Roles input box:

![image](https://user-images.githubusercontent.com/300/101226219-9d53eb80-3648-11eb-846e-df0eecb442ba.png)

Edit your original user to have the role "admin":

![image](https://user-images.githubusercontent.com/300/101226249-ba88ba00-3648-11eb-8e83-7b4d17822442.png)

Be sure to accept the invite for your new user and set a password so that you can actually log in as them.

If all went well, you should be able to log in as either user with no change in the functionality between them—both can access http://localhost:8910/admin/posts Log in as your moderator user and go there now so we can verify that we get booted out once we add some authorization rules.

### Roles in Routes

The easiest form of RBAC involves locking down entire routes. Let's add one so that only admins can see the admin pages.

In the Router simply add a `role` prop and pass it the name of the role that should be allowed. This prop also [accepts an array](/cookbook/role-based-access-control-rbac#how-to-protect-a-route) if more than one role should have access.

```javascript{12}
// web/src/Routes.js

import { Router, Route, Private } from '@redwoodjs/router'

const Routes = () => {
  return (
    <Router>
      <Route path="/contact" page={ContactPage} name="contact" />
      <Route path="/blog-post/{id:Int}" page={BlogPostPage} name="blogPost" />
      <Route path="/about" page={AboutPage} name="about" />
      <Route path="/" page={HomePage} name="home" />
      <Private unauthenticated="home" role="admin">
        <Route path="/admin/posts/new" page={NewPostPage} name="newPost" />
        <Route path="/admin/posts/{id:Int}/edit" page={EditPostPage} name="editPost" />
        <Route path="/admin/posts/{id:Int}" page={PostPage} name="post" />
        <Route path="/admin/posts" page={PostsPage} name="posts" />
      </Private>
      <Route notfound page={NotFoundPage} />
    </Router>
  )
}

export default Routes
```

When you save that change the browser should refresh and you'll be sent back to the homepage. Log out and back in as the admin user and you should still have access.

### Roles in Components

Locking down a whole page is easy enough, but what about individual functionality within a page or component?

Redwood provides a `hasRole()` function you can get from the `useAuth()` hook which returns `true` or `false` depending on whether the logged in user has the given role. Let's try it out by adding a `Delete` button when a moderator is viewing a blog post's comments:

```javascript{3,12-17,28-36}
// web/src/components/Comment/Comment.js

import { useAuth } from '@redwoodjs/auth'

const formattedDate = (datetime) => {
  const parsedDate = new Date(datetime)
  const month = parsedDate.toLocaleString('default', { month: 'long' })
  return `${parsedDate.getDate()} ${month} ${parsedDate.getFullYear()}`
}

const Comment = ({ comment }) => {
  const { hasRole } = useAuth()
  const moderate = () => {
    if (confirm('Are you sure?')) {
      // TODO: delete a comment
    }
  }

  return (
    <div className="relative bg-gray-200 p-8 rounded-lg">
      <header className="flex justify-between">
        <h2 className="font-semibold text-gray-700">{comment.name}</h2>
        <time className="text-xs text-gray-500" dateTime={comment.createdAt}>
          {formattedDate(comment.createdAt)}
        </time>
      </header>
      <p className="text-sm mt-2">{comment.body}</p>
      {hasRole('moderator') && (
        <button
          type="button"
          onClick={moderate}
          className="absolute bottom-2 right-2 bg-red-500 text-xs rounded text-white px-2 py-1"
        >
          Delete
        </button>
      )}
    </div>
  )
}

export default Comment
```

> **What about the tests you hypocrite??**
>
> We'd love to say this is an instance where leaving the exercise for the student is a way to achieve a deeper understanding of the techniques we've learned, but really it's because this second tutorial is already really long and we could all use a break.
>
> But really, you should add tests around this functionality!

So if the user has the "moderator" role, render the delete button. If you log out and back in as the admin, or if you log out completely, you'll see the delete button go away. When logged out (that is, `curentUser === null`) `hasRole()` will always return `false`.

![image](https://user-images.githubusercontent.com/300/101229168-c75edb00-3653-11eb-85f0-6eb61af7d4e6.png)

What should we put in place of the TODO? A GraphQL mutation that deletes a comment, of course. We'll need a new service function, GraphQL endpoint on the api-side and a new GraphQL and `useMutation()` call on the web-side.

Thanks to our forward-thinking earlier we already have a `deleteComment()` in our comments service, so we just need a GraphQL mutation to get the web-side access:

```graphql{3-5,9}
// api/src/graphql/comments.sdl.js

type Mutation {
  createComment(input: CreateCommentInput!): Comment!
  deleteComment(id: Int!): Comment!
}
```

`deleteComment` will be given a single argument, the ID of the comment to delete, and it's required. A common pattern is to return the record that was just deleted in case you wanted to notify the user or some other system about the details of the thing that was just removed, so we'll do that here as well. But, you could just as well return `null`.

And thanks to the nice encapsultation of our **Comment** component we can make all the required web-site changes in this one component:

```javascript{3-5,13-19,23-30,33-35}
// web/src/components/Comment/Comment.js

import { useAuth } from '@redwoodjs/auth'
import { useMutation } from '@redwoodjs/web'
import { QUERY as CommentsQuery } from 'src/components/CommentsCell'

const formattedDate = (datetime) => {
  const parsedDate = new Date(datetime)
  const month = parsedDate.toLocaleString('default', { month: 'long' })
  return `${parsedDate.getDate()} ${month} ${parsedDate.getFullYear()}`
}

const DELETE = gql`
  mutation DeleteCommentMutation($id: Int!) {
    deleteComment(id: $id) {
      postId
    }
  }
`

const Comment = ({ comment }) => {
  const { hasRole } = useAuth()
  const [deleteComment] = useMutation(DELETE, {
    refetchQueries: [
      {
        query: CommentsQuery,
        variables: { postId: comment.postId },
      },
    ],
  })
  const moderate = () => {
    if (confirm('Are you sure?')) {
      deleteComment({
        variables: { id: comment.id },
      })
    }
  }

  return (
    <div className="relative bg-gray-200 p-8 rounded-lg">
      <header className="flex justify-between">
        <h2 className="font-semibold text-gray-700">{comment.name}</h2>
        <time className="text-xs text-gray-500" dateTime={comment.createdAt}>
          {formattedDate(comment.createdAt)}
        </time>
      </header>
      <p className="text-sm mt-2">{comment.body}</p>
      {hasRole('moderator') && (
        <button
          type="button"
          onClick={moderate}
          className="absolute bottom-2 right-2 bg-red-500 text-xs rounded text-white px-2 py-1"
        >
          Delete
        </button>
      )}
    </div>
  )
}

export default Comment
```

Click "Delete" (as a moderator) and the comment should be removed!

Having a role like "admin" implies that they can do everything...shouldn't they be able to delete comments as well? Right you are! There are two things we can do here:

* Add "admin" to the list of roles in the `hasRole` function call
* Additionally add the "moderator" role to the list of roles that the admin has in Netlify Identity

By virtue of the name "admin" it really feels like someone should only have that one single roll and be able to do everything. So in this case it feels better to add "admin" to `hasRole()`.

If you wanted to be more fine-grained with your roles then maybe the "admin" role should really be called "author". That way it makes it clear they only author posts, and if you want someone to be able to do both actions, then you explicity give them the "moderator" role in addition to "author."

## Wrapping up

TBD