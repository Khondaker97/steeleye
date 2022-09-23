# Front End Developer Intern assignment

1. List component memoise the WrappedListComponent that means it won't be re-rendered
   with other components in its parent component if the recieved items prop hasn't changed.

2. As we looked at the code, we found several issuses here.

- Each time items prop change its state, it is intended to set the selectedIndex null
  but within the useEffect setSelectedIndex itself the state, not the setter function.
  As we know, destructuring useState hook gives us the state first and the second one is a setter
  function which have the record of current state. we can write selectedIndex(null) within the
  useEffect hook but it is convention to write useState the following way.

```
    const [selectedIndex, setSelectedIndex] = useState();
```

Hence, we need to change several lines of code here.

- Now, the items prop is an array of object that has a property 'text'.
  Therefore, PropTypes of items should be otherwise than the given one.
  PropTypes.arrayOf instead of PropTypes.array and PropTypes.shape in place of PropTypes.shapeOf.

```
    WrappedListComponent.propTypes = {
            items: PropTypes.arrayOf(PropTypes.shape({
                text: PropTypes.string.isRequired,
            })),
        };

```

- Items prop is null by default. So, we should use optional chaining before mapping through
  items so that if no items is provided to List component, it won't crash the app rather render it
  when items are available. Moreover, each item in SingleListItem should have a boolean value for
  isSelected prop rather giving a number through selectedIndex as WrappedSingleListItem expected
  the boolean type of isSelected. There is a big but using a state for isSelected in parent component.
  I will clear it next.

- When we look at the WrappedSingleListItem, it is memoised as well as intend to not re-render
  when parent renders. And only the selected child's background will change via onClick. But onclick
  generates new referance of function in the parent causes all the child re-render. consequently, we
  have to wrap the handleClick function with useCallback hook. Again, that's not preventing child's re-render.
  Beacuse we got the index with map function,so, we have to recieve the index parameter from child component,
  not from the parent.

```
{items?.map((item, index) => (
    <SingleListItem
        key={index}
        onClickHandler={handleClick}
        text={item.text}
        index={index}
    />
    ))}
```

Where's the isSelected prop? It's coming...

- Now, all the SingleListItem has isSelected prop. if we click on a SingleListItem then all the
  child's background has changed. Why is that? Cause all these components share same state. Thus,
  when isSelected state changes it changes for all. Therefore, removing isSelected from the parent
  component we set it in the child component. Now, every child has isSelected state, due to that,
  we can change background of a particular child component.

### Optimise code

```
import React, { useState, useEffect, memo, useCallback } from "react";
import PropTypes from "prop-types";

// Single List Item
const WrappedSingleListItem = ({ index, onClickHandler, text }) => {
  console.log("child render");
  const [isSelected, setIsSelected] = useState(false);
  console.log(isSelected);
  return (
    <li
      style={{
        backgroundColor: isSelected ? "green" : "red",
        marginBottom: "2rem",
      }}
      onClick={() => {
        onClickHandler(index);
        setIsSelected((prev) => !prev);
      }}
    >
      {text}
    </li>
  );
};

WrappedSingleListItem.propTypes = {
  index: PropTypes.number,
  isSelected: PropTypes.bool,
  onClickHandler: PropTypes.func.isRequired,
  text: PropTypes.string.isRequired,
};

const SingleListItem = memo(WrappedSingleListItem);

// List Component
const WrappedListComponent = ({ items }) => {
  const [selectedIndex, setSelectedIndex] = useState();

  useEffect(() => {
    setSelectedIndex(null);
  }, [items]);

  const handleClick = useCallback((index) => {
    setSelectedIndex(index);
  }, []);

  return (
    <ul style={{ textAlign: "left" }}>
      {items?.map((item, index) => (
        <SingleListItem
          key={index}
          onClickHandler={handleClick}
          text={item.text}
          index={index}
        />
      ))}
    </ul>
  );
};

WrappedListComponent.propTypes = {
  items: PropTypes.arrayOf(
    PropTypes.shape({
      text: PropTypes.string.isRequired,
    })
  ),
};

WrappedListComponent.defaultProps = {
  items: null,
};

const List = memo(WrappedListComponent);

export default List;

```
