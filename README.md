# Thinking in React <!-- omit in toc -->

When you build a user interface with React, you will first break it apart into pieces called components. Then, you will describe the different visual states for each of your components. Finally, you will connect your components together so that the data flows through them. In this tutorial, we’ll guide you through the thought process of building a searchable product data table with React.

# Table of Contents <!-- omit in toc -->

- [Start with a mockup](#start-with-a-mockup)
- [Step 1: Break the UI into a component hierarchy](#step-1-break-the-ui-into-a-component-hierarchy)
- [Step 2: Build a static version in React](#step-2-build-a-static-version-in-react)
- [Step 3: Find the minimal but complete representation of UI state](#step-3-find-the-minimal-but-complete-representation-of-ui-state)
- [Step 4: Identify where your state should live](#step-4-identify-where-your-state-should-live)
- [Step 5: Add inverse data flow](#step-5-add-inverse-data-flow)

# Start with a mockup

Imagine that you already have a JSON API and a mockup from a designer.

The JSON API returns some data that looks like this:

```json
// prettier-ignore
[
  { category: "Fruits", price: "$1", stocked: true, name: "Apple" },
  { category: "Fruits", price: "$1", stocked: true, name: "Dragonfruit" },
  { category: "Fruits", price: "$2", stocked: false, name: "Passionfruit" },
  { category: "Vegetables", price: "$2", stocked: true, name: "Spinach" },
  { category: "Vegetables", price: "$4", stocked: false, name: "Pumpkin" },
  { category: "Vegetables", price: "$1", stocked: true, name: "Peas" }
]
```

The mockup looks like this:

![alt text](doc_images/image01.png)

To implement a UI in React, you will usually follow the same five steps:

# Step 1: Break the UI into a component hierarchy

Start by drawing boxes around every component and subcomponent in the mockup and naming them.

You can think about splitting up a design into components in this way:

> A component should ideally only be concerned with one thing. If it ends up growing, it should be decomposed into smaller subcomponents.

If your JSON is well-structured, you’ll often find that it naturally maps to the component structure of your UI. Separate your UI into components, where each component matches one piece of your data model.

There are five components on this screen:

![alt text](doc_images/image02.png)

1. `FilterableProductTable` (grey) contains the entire app.
2. `SearchBar` (blue) receives the user input.
3. `ProductTable` (lavender) displays and filters the list according to the user input.
4. `ProductCategoryRow` (green) displays a heading for each category.
5. `ProductRow` (yellow) displays a row for each product.

If you look at `ProductTable` (lavender), you’ll see that the table header (containing the “Name” and “Price” labels) isn’t its own component. This is a matter of preference. For this example, it is a part of `ProductTable` because it appears inside the `ProductTable`’s list. However, if this header grows to be complex (e.g., if you add sorting), you can move it into its own `ProductTableHeader` component.

Now that you’ve identified the components in the mockup, arrange them into a hierarchy:

- FilterableProductTable
  - SearchBar
  - ProductTable
    - ProductCategoryRow
    - ProductRow

# Step 2: Build a static version in React

Now, it’s time to implement your app. It’s often easier to build the static version first and add interactivity later. Building a static version requires a lot of typing and no thinking, but adding interactivity requires a lot of thinking and not a lot of typing.

To build a static version of your app that renders your data model, you’ll want to build components that reuse other components and pass data using props. Props are a way of passing data from parent to child. (don’t use state at all to build this static version. State is reserved only for interactivity, that is, data that changes over time.)

You can either build “top down” by starting with building the components higher up in the hierarchy (like `FilterableProductTable`) or “bottom up” by working from components lower down (like `ProductRow`). In simpler examples, it’s usually easier to go top-down, and on larger projects, it’s easier to go bottom-up.

First, create a React app by following [this link](https://github.com/ttanvirr/react-starter) (Only basic setup with javascript)

In the `App.js` file, replace everything with the following:

```jsx
function SearchBar() {
  return (
    <form>
      <input type="text" placeholder="Search..." />
      <br />
      <label>
        <input type="checkbox" /> Only show products in stock
      </label>
    </form>
  )
}

function ProductCategoryRow({ category }) {
  return (
    <tr>
      <th colSpan="2">{category}</th>
    </tr>
  )
}
function ProductRow({ product }) {
  return (
    <tr>
      <td>
        {product.stocked ? (
          product.name
        ) : (
          <span style={{ color: "red" }}>{product.name}</span>
        )}
      </td>
      <td align="right">{product.price}</td>
    </tr>
  )
}

function ProductTable({ products }) {
  let lastCategory = null
  const rows = []

  products.forEach((product) => {
    if (product.category !== lastCategory) {
      rows.push(
        <ProductCategoryRow
          category={product.category}
          key={product.category}
        />,
      )
    }

    rows.push(<ProductRow product={product} key={product.name} />)

    lastCategory = product.category
  })

  return (
    <table>
      <thead>
        <tr>
          <th>Name</th>
          <th>Price</th>
        </tr>
      </thead>

      <tbody>{rows}</tbody>
    </table>
  )
}
function FilterableProductTable({ products }) {
  return (
    <div>
      <SearchBar />
      <ProductTable products={products} />
    </div>
  )
}

const PRODUCTS = [
  { category: "Fruits", price: "$1", stocked: true, name: "Apple" },
  { category: "Fruits", price: "$1", stocked: true, name: "Dragonfruit" },
  { category: "Fruits", price: "$2", stocked: false, name: "Passionfruit" },
  { category: "Vegetables", price: "$2", stocked: true, name: "Spinach" },
  { category: "Vegetables", price: "$4", stocked: false, name: "Pumpkin" },
  { category: "Vegetables", price: "$1", stocked: true, name: "Peas" },
]

function App() {
  return <FilterableProductTable products={PRODUCTS} />
}

export default App
```

Now, you have a library of reusable components that render your data model. The component at the top of the hierarchy (`FilterableProductTable`) will take your data model as a prop. This is called one-way data flow because the data flows down from the top-level component to the ones at the bottom of the tree.

# Step 3: Find the minimal but complete representation of UI state

To make the UI interactive, you need to let users change your underlying data model. You will use _state_ for this.

Think of `state` as the minimal set of changing data that your app needs to remember. The most important principle for structuring state is to keep it _DRY (Don’t Repeat Yourself)_. Figure out the absolute minimal representation of the state your application needs and compute everything else on-demand. For example, if you’re building a shopping list, you can store the items as an array in state. If you want to also display the number of items in the list, don’t store the number of items as another state value—instead, read the length of your array.

Now think of all of the pieces of data in this example application:

- The original list of products
- The search text the user has entered
- The value of the checkbox
- The filtered list of products

Which of these are state? Identify the ones that are not:

- Does it remain unchanged over time? If so, it isn’t state.
- Is it passed in from a parent via props? If so, it isn’t state.
- Can you compute it based on existing state or props in your component? If so, it definitely isn’t state!

What’s left is probably state.

Let’s go through them one by one again:

- The original list of products is passed in as props, so it’s not state.
- The search text seems to be state since it changes over time and can’t be computed from anything.
- The value of the checkbox seems to be state since it changes over time and can’t be computed from anything.
- The filtered list of products isn’t state because it can be computed by taking the original list of products and filtering it according to the search text and value of the checkbox.

This means only the search text and the value of the checkbox are state!

> [!NOTE]
> **Props vs State**
>
> There are two types of “model” data in React: props and state:
>
> - _Props are like arguments_ you pass to a function. They let a parent component pass data to a child component and customize its appearance. For example, a `Form` can pass a `color` prop to a `Button`.
> - _State is like a component’s memory_. It lets a component keep track of some information and change it in response to interactions. For example, a `Button` might keep track of `isHovered` state.
>
> Props and state are different, but they work together. A parent component will often keep some information in state (so that it can change it), and pass it down to child components as their props.

# Step 4: Identify where your state should live

After identifying your app’s minimal state data, you need to identify which component is responsible for changing this state, or owns the state. Remember: React uses one-way data flow, passing data down the component hierarchy from parent to child component. You can figure out 'which component should own what state' by following these steps!

For each piece of state in your application:

- Identify every component that renders something based on that state.
- Find their closest common parent component.
- Decide where the state should live:
  - Often, you can put the state directly into their common parent.
  - You can also put the state into some component above their common parent.
  - If you can’t find a component where it makes sense to own the state, create a new component solely for holding the state and add it somewhere in the hierarchy above the common parent component.

In the previous step, you found two pieces of state in this application: the search input text, and the value of the checkbox. In this example, they always appear together, so it makes sense to put them into the same place.

Now let’s run through our strategy for them:

1. **Identify components that use state:**
   - `ProductTable` needs to filter the product list based on that state (search text and checkbox value).
   - `SearchBar` needs to display that state (search text and checkbox value).
2. Find their common parent: The first parent component both components share is `FilterableProductTable`.
3. Decide where the state lives: We’ll keep the filter text and checked state values in `FilterableProductTable`.

Add state to the component with the `useState()` Hook. Hooks are special functions that let you “hook into” React. Add two state variables at the top of `FilterableProductTable` and specify their initial state:

`App.jsx`

```jsx
import { useState } from 'react';

function FilterableProductTable({ products }) {
  const [filterText, setFilterText] = useState('');
  const [inStockOnly, setInStockOnly] = useState(false);

  return (
    // ...
  )
```

Then, pass `filterText` and `inStockOnly` to `ProductTable` and `SearchBar` as props:

```jsx
function FilterableProductTable({ products }) {
  // ...

  return (
    <div>
      <SearchBar filterText={filterText} inStockOnly={inStockOnly} />
      <ProductTable
        products={products}
        filterText={filterText}
        inStockOnly={inStockOnly}
      />
    </div>
  )
}
```

First, recieve the props in `SearchBar` and use them:

```jsx
function SearchBar({ filterText, inStockOnly }) {
  return (
    <form>
      <input type="text" value={filterText} placeholder="Search..." />
      <br />
      <label>
        <input type="checkbox" checked={inStockOnly} /> Only show products in
        stock
      </label>
    </form>
  )
}
```

Next, recieve the props in `ProductTable` to filter the product list:

```jsx
function ProductTable({ products, filterText, inStockOnly }) {
  let lastCategory = null
  const rows = []

  products.forEach((product) => {
    // If name doesn't contain the filterText, return
    if (product.name.toLowerCase().indexOf(filterText.toLowerCase()) === -1)
      return

    // If inStockOnly is checked, and product.stocked is false, return
    if (inStockOnly && !product.stocked) return

    if (product.category !== lastCategory) {
      rows.push(
        <ProductCategoryRow
          category={product.category}
          key={product.category}
        />,
      )
    }

    rows.push(<ProductRow product={product} key={product.name} />)

    lastCategory = product.category
  })

  return (
    //...
  )
}
```

You can start seeing how your application will behave. Edit the `filterText` initial value from `useState('')` to `useState('fruit')`. You’ll see both the search input text and the table update.

Notice that editing the form doesn’t work yet. There is a console error in the browser's console:

```
You provided a `value` prop to a form field without an `onChange` handler. This will render a read-only field.
```

You haven’t added any code to respond to the user actions like typing yet. This will be your final step.

# Step 5: Add inverse data flow

Currently your app renders correctly with props and state flowing down the hierarchy. But to change the state according to user input, you will need to support data flowing the other way: the `form` components deep in the hierarchy need to update the state in `FilterableProductTable`.

React makes this data flow explicit. By writing `<input value={filterText} />`, you’ve set the value prop of the input to always be equal to the `filterText` state passed in from `FilterableProductTable`. Since `filterText` state is never set, the input never changes.

You want to make it so whenever the user changes the form inputs, the state updates to reflect those changes. The state is owned by `FilterableProductTable`, so only it can call `setFilterText` and `setInStockOnly`. To let `SearchBar` update the `FilterableProductTable`’s state, you need to pass these functions down to `SearchBar`:

```jsx
function FilterableProductTable({ products }) {
  const [filterText, setFilterText] = useState('');
  const [inStockOnly, setInStockOnly] = useState(false);

  return (
    <div>
      <SearchBar
        filterText={filterText}
        inStockOnly={inStockOnly}
        onFilterTextChange={setFilterText}
        onInStockOnlyChange={setInStockOnly} />
```

Inside the SearchBar, you will add the onChange event handlers and set the parent state from them:

```jsx
function SearchBar({
  filterText,
  inStockOnly,
  onFilterTextChange,
  onInStockOnlyChange
}) {
  return (
    <form>
      <input
        type="text"
        value={filterText}
        placeholder="Search..."
        onChange={(e) => onFilterTextChange(e.target.value)}
      />
      <label>
        <input
          type="checkbox"
          checked={inStockOnly}
          onChange={(e) => onInStockOnlyChange(e.target.checked)}
```

Now the application fully works!
