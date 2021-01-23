<!-- Love Jest? Please consider supporting our collective: 👉  https://opencollective.com/jest/donate -->

## 🐛 Bug Report

Jest's timer mocks not work properly.
`setTimeout` not work in jest.

### card.js
```js
import React, { useEffect } from "react";

export default function Card(props) {
  useEffect(() => {
    const timeoutID = setTimeout(() => {
      props.onSelect(null);
    }, 5000);

    return () => {
      clearTimeout(timeoutID);
    };
  }, [props.onSelect]);

  return [1, 2, 3, 4].map((choice) => (
    <button
      key={choice}
      data-testid={choice}
      onClick={() => props.onSelect(choice)}
    >
      {choice}
    </button>
  ));
}
```

### card.test.js

```js
// card.test.js

import React from "react";
import { render, unmountComponentAtNode } from "react-dom";
import { act } from "react-dom/test-utils";

import Card from "./card";

jest.useFakeTimers();

let container = null;
beforeEach(() => {
  // setup a DOM element as a render target
  container = document.createElement("div");
  document.body.appendChild(container);
});

afterEach(() => {
  // cleanup on exiting
  unmountComponentAtNode(container);
  container.remove();
  container = null;
});

it("should select null after timing out", () => {
  const onSelect = jest.fn();
  act(() => {
    render(<Card onSelect={onSelect} />, container);
  });

  // move ahead in time by 100ms
  act(() => {
    jest.advanceTimersByTime(100);
  });
  expect(onSelect).not.toHaveBeenCalled();

  // and then move ahead by 5 seconds
  act(() => {
    jest.advanceTimersByTime(5000);
  });
  expect(onSelect).toHaveBeenCalledWith(null);
});

it("should cleanup on being removed", () => {
  const onSelect = jest.fn();
  act(() => {
    render(<Card onSelect={onSelect} />, container);
  });
  act(() => {
    jest.advanceTimersByTime(100);
  });
  expect(onSelect).not.toHaveBeenCalled();

  // unmount the app
  act(() => {
    render(null, container);
  });
  act(() => {
    jest.advanceTimersByTime(5000);
  });
  expect(onSelect).not.toHaveBeenCalled();
});

it("should accept selections", () => {
  const onSelect = jest.fn();
  act(() => {
    render(<Card onSelect={onSelect} />, container);
  });

  act(() => {
    container
      .querySelector("[data-testid='2']")
      .dispatchEvent(new MouseEvent("click", { bubbles: true }));
  });

  expect(onSelect).toHaveBeenCalledWith(2);
});

```

### Terminal Output
```
 FAIL  src/card.test.js
  ● should select null after timing out

    expect(jest.fn()).toHaveBeenCalledWith(...expected)

    Expected: null

    Number of calls: 0

      39 |     jest.advanceTimersByTime(5000);
      40 |   });
    > 41 |   expect(onSelect).toHaveBeenCalledWith(null);
         |                    ^
      42 | });
      43 | 
      44 | it("should cleanup on being removed", () => {

      at Object.<anonymous> (src/card.test.js:41:20)

 FAIL  src/contact.test.js
  ● Test suite failed to run

    ENOENT: no such file or directory, open '/Volumes/Data/WORK/devHabit/test-skillup/src/contact.test.js'

      at runTestInternal (node_modules/jest-runner/build/runTest.js:202:27)

 PASS  src/App.test.js

Test Suites: 2 failed, 1 passed, 3 total
Tests:       1 failed, 3 passed, 4 total
Snapshots:   0 total
Time:        2.647 s
Ran all test suites.

Watch Usage: Press w to show more.
```

## To Reproduce

- Create new react app by using `create-react-app` package.
- Follow the `Timers` part in [`Testing Recipes`](https://reactjs.org/docs/testing-recipes.html) chapter from React Docs page.
- Run test. `yarn test`

## Expected behavior

All the tests should be passed.

## Link to repl or repo (highly encouraged)

Please check this [repository](https://github.com/DarkPonyBee/jest-timer-issue).

## envinfo


```
System:
    OS: macOS 11.2
    CPU: (4) x64 Intel(R) Core(TM) i3-9100 CPU @ 3.60GHz
  Binaries:
    Node: 15.2.0 - /usr/local/bin/node
    Yarn: 1.22.10 - /usr/local/bin/yarn
    npm: 7.0.10 - /usr/local/bin/npm
```
