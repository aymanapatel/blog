---
title: "Snapshot testing"
seoTitle: "Snapshot testing"
seoDescription: "We have been writing unit test for UI without actually thinking the environment they run in. We assume that it will catch issues before they surface to cust"
datePublished: Sun Apr 27 2025 07:40:56 GMT+0000 (Coordinated Universal Time)
cuid: cm9zcbfu5000109lb27bof5mn
slug: snapshot-testing
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1745739615114/7bb4a528-45de-4c2c-bba1-1679225a0d9d.jpeg
tags: testing, snapshot-testing

---

# Preface: Why Unit test fails

We have been writing unit test for UI without actually thinking the environment they run in. We assume that it will catch issues before they surface to customers, but sadly there needs to be a lot of introspection if the test is written for the sake of writing it, or improving the applications we develop.  
Sometimes, going for 100% code coverage is not the right metric. It is necessary to have the highest amount of code coverage, but it is certainly not **sufficient**!

## Act 1 \[2014 \]: Enzymes and Protractor

Testing is a necessary part of software. Especially in UI, when the bugs are user-facing. A weird or ugly looking UI can set off a very bad impression. But the environments we typically use for testing do not usually resemble the actual UI, they run in environments

We used to have things like Enzyme and Protractor. Enzyme was built by Airbnb with jQuery-like API to interface with UI components.  
These sort of architecture wherein meant you needed to test for implementation.  
For example, if you want to test for `useState`, you

```javascript
import React, { useState, useEffect } from 'react';

function UserProfile({ userId }) {
  const [user, setUser] = useState(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);

  useEffect(() => {
    const fetchUser = async () => {
      try {
        setLoading(true);
        const response = await fetch(`https://api.example.com/users/${userId}`);
        if (!response.ok) throw new Error('Failed to fetch user');
        const data = await response.json();
        setUser(data);
      } catch (err) {
        setError(err.message);
      } finally {
        setLoading(false);
      }
    };

    fetchUser();
  }, [userId]);

  if (loading) return <div className="loading-state">Loading...</div>;
  if (error) return <div className="error-state">Error: {error}</div>;
  
  return (
    <div className="user-profile">
      <h2>{user.name}</h2>
      <p>Email: {user.email}</p>
    </div>
  );
}

export default UserProfile;
```

## Enzyme Implementation Testing

Enzyme tests often access component internals directly:

```javascript
import { mount } from 'enzyme';
describe('UserProfile component with Enzyme', () => {
  // Mock fetch globally
  const mockSuccessResponse = { id: 1, name: 'John Doe', email: 'john@example.com' };
  const mockFetchPromise = Promise.resolve({
    ok: true,json: () => Promise.resolve(mockSuccessResponse),
  });
  
  beforeEach(() => {global.fetch = jest.fn().mockImplementation(() => mockFetchPromise);});

  afterEach(() => {global.fetch = undefined;});

  it('shows loading state initially', () => {
    const wrapper = mount(<UserProfile userId="1" />);
    
    // Implementation detail: checking for specific class and loding state
    expect(wrapper.find('.loading-state').exists()).toBe(true);
    expect(wrapper.state('loading')).toBe(true);
    // Implementation detail: wait for component to update
    await mockFetchPromise;
    wrapper.update();
    
    // Implementation detail: examining component internals
    expect(wrapper.state('user')).toEqual(mockSuccessResponse);
    expect(wrapper.state('loading')).toBe(false);
    expect(wrapper.find('h2').text()).toBe('John Doe');
    expect(wrapper.find('.user-profile').exists()).toBe(true);
  });


  it('updates when userId prop changes', async () => {
    const wrapper = mount(<UserProfile userId="1" />);
    
    // Implementation detail: checking fetch call details
    expect(global.fetch).toHaveBeenCalledWith('https://api.example.com/users/1');
    
    // Update prop and verify new fetch
    wrapper.setProps({ userId: '2' });
    expect(global.fetch).toHaveBeenCalledWith('https://api.example.com/users/2');
  });
});
```

This is testing very specific details in a very specific environment. What if we had to throttle the API.  
It is possible, but bloody damn difficult and manual.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1745656260315/ff74f5e6-877f-4de4-8d5d-50483bf39f5b.png align="center")

%[https://gist.github.com/aymanapatel/42a1775309c72cc4ae629b839bb8b6ff] 

We’ll compare this code and its complexity when using Testing library with Mock service worker. But if you see some snippets like `jest.advanceTimersByTime(1000);` and below:

```javascript
      const delay = requestCount === 1 ? 500 : 1500; // First request faster, second slower
      const mockUserData = { id: requestCount, name: 'John Doe', email: 'john@example.com' };
      return new Promise(resolve => {
        setTimeout(() => {
          resolve({
            ok: true,
            json: () => Promise.resolve(mockUserData)
          });
        }, delay);
```

You realize how much setup code is required. This causes 2 issues:

1. Less inclination to handle these kind of scenarios as there is too much boilerplate.
    
2. If adding, maintaining the test cases becomes a hassle.
    

And the UI that breaks is less to do with the bug in an iterative run but more to do with dynamic nature of web such as slow APIs, inconsistent state when working across components, CSS overrides .etc.

## Act 2 \[2019\]: Testing library

Testing library brought a different paradigm. It focuesed on writing integration tests with accessibility DOM tree at the forefront.  
Kent C. Odds (one of maintainers of RTL) rooted for writing integration tests and less unit and mocky tests. It makes sense. You need to see the Return on Investment of writing test. A 95% coverage does not have a order-of-magnitude improvement over 90% in spite of having an order-of-magnitude in writing + maintaining test. Also bugs arise at the integration level such as

1. Network requests
    
2. Incorrect app state
    
3. CSS overrides
    
4. Much more
    

And no amount of meeting a conditional code coverage criteria or hand picking state variables is going to improve your “test coverage”.

Looking at the throttling example, with RTL it becomes much simpler.

The following uses Mock Service Worker which is a handy tool to mock API request, response, response times and works with Testing Library, Storybook, Playwright etc.

%[https://gist.github.com/aymanapatel/e3c6aae84371480e4984c75f594ce9fb] 

1. RTL: 115 lines
    
2. Enzyme: 153 Lines
    

But if you look at the individual test case it is almost **double** in the case of **Enzyme**.

Also RTL uses MSW that handles most of real-world simulation of APIs.

Simplicity and predictability in test cases makes the test suite for robust than following a made up metric (Please read [Goodhart’s law](https://en.wikipedia.org/wiki/Goodhart%27s_law))

# Snapshot testing

In the spirit of not chasing made up metrics without a strong case for “100%” coverage, we now come to a particular type of testing that is very much needed in UI.

## Snapshot Unit tests

> All the code can be found [here](https://gitlab.com/aymanapatelgroup/blog-snapshot-testing)

### Example 1: Vitest JSDOM mode

### Simple working example

Here we have a Button whose styles are determined inside the component. A property `primary` is passed that does calculation on what styles need to be rendered.

```javascript
// Button.jsx
import React from 'react';

const Button = ({ text, primary = false }: {text: string, primary: boolean}) => {
  // The visual regression bug: 
  // When primary is true, it should have a blue background,
  // but the color is incorrectly set to a very similar shade that looks almost identical
  // in the snapshot serialization but is visually different
  const style = {
    // other styles
    backgroundColor: primary ? '#0070f3' : '#f0f0f0', // Correct color for primary
    color: primary ? 'white' : 'black',
  };
  const overideStyle = {
    ...style,
    backgroundColor: primary ? '#0071f4' : '#f0f0f0', // Slightly different blue that passes in snapshots
    padding: primary ? '8px 16px' : '10px 20px', 
  };

  return (
    <button 
      style={overideStyle}
      className={`button ${primary ? 'primary' : 'secondary'} parent-override`}
    >
      {text}
    </button>
  );
};

export default Button;
```

This test passes:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1745737243373/45486972-8491-4913-9d40-de10a1d4b3da.png align="center")

Now if we change the color in the button component; then we get a test fail.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1745737720465/9189949c-0442-43b0-b371-1394cce15520.png align="center")

  
Note that it is a trivial example, but in real world these could be coming from CSS variables that can cause this bug to be caught. Note that JSDOM does not work with inherited styles which will be covered later.

### Simple non-working example

Consider the example of CSS overrides though inteheritance. These are common in real systems as we highlight further.  
This is our initial code:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1745736447209/4b7bc02b-fba3-4a93-b688-117f19c9bf25.png align="center")

With the CSS:

```css
.primary {
    background-color: green;
    border-radius: 4px;
    padding: 8px 16px;
    color: white;
}
```

  
  
And the resulting screen:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1745736479810/df82ec3c-bcc1-49ed-8645-330631ac4cfd.png align="center")

  
Now let me add a bad CSS `!important`. You might say that this is bad practice, but in real life, there is always someone using this to make their colors pop. Could be your design system, your cookie banner, or an old CSS file overriding this. So it is a common occurrence in real world. So we need to have a test that checks this.  

```css
.primary {
    background-color: green;
    border-radius: 4px;
    padding: 8px 16px;
    color: white;
}


button {
    background-color: red !important;
}
```

This is the resultant screenshot:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1745736678960/6f373d70-5706-429f-94b8-d595db8193f1.png align="center")

You would expect the test to fail, but behold it still pass.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1745736814324/29ae4058-1a60-4e05-9828-95e1395d6672.png align="center")

What is worse than a failing test, is a test that passes in spite of having a bug. Why did this happen:

1. The `*.snap` file compares the DOM of that particular element. The file is the same, hence it passes.
    

2. JSDOM cannot do compute styles from one DOM element to its child.
    

> Now, do you trust your tests?! Now do you think 100% code coverage is enough!

### Case against JSDOM

1. JSDOM is just a poor HTML implementation
    

In the we can see that JSDOM relies on HTML. This works well for simple snapshots, but when there are dynamic sites such as animations, CSS calculations, responsive sites etc; then the snapshots start to fall apart.

2. JSDOM has a list of CSS features that it does not natively support:
    

* Media Queries: This means you can test things that are dependent on the responsive designs.
    
* CSS styles can not inherited (styles coming from parent DOM element) as `getComputedStyle` is not inherited in Jest. Look at this [Github issue](https://github.com/jsdom/jsdom/issues/2160)
    
    * ```javascript
        const dom = new JSDOM(`
        <style>
          body {
            color: Red;
          }
        </style>hello world
        <div class="divgroup" id="d1" 
             style="padding-right: 5px;">
             Content for div 1
        </div>
        `);
        // Found
        dom.window.getComputedStyle(
        dom.window.document.querySelector('#d1'))
        
        // Cannot be found as the `body` style is not inhertiable
        dom.window.getComputedStyle(
        dom.window.document.querySelector('#d1')).getPropertyValue('color'))
        ```
        
* Does not support injection stylesheets inside JSDOM environment. Many runtime CSS-in-JS libraries such as Styled components hence do not work nicely with the JSDOM environment. For more details, read this [Github issue](https://github.com/testing-library/jest-dom/issues/116)
    

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1745734300171/31029499-c965-4d93-8ae5-4aad2839a048.png align="center")

3. JSDOM requires fetch overrides (monkey-patching)
    
    1. You have to use libraries like [jest-fetch-mock](https://www.npmjs.com/package/jest-fetch-mock) or nock. Well they do get the job done, but it is a monkeypatch or a bandaid. It cannot provide easy and fine grained network simulation like slow requests or internet disconnection.
        

### Example 2: Playwright Snapshot tests

In order to combat the issues we typically see in non-browser environments such as JSDOM, it is better to use an environment that is as close to user as possible; which in case would be the actual browser.

Playwright has become the de-facto tool for E2E test. Unlike other tools such as Selenium and Cypress which is has many limitations due to the architecture underneath using Selenium Webdriver and Cypress browser respectively; Playwright is becoming more mainstream due to its usage Chrome dev tools.

For Selenium, since it uses Webdriver which has a lot of translation/hops between the test and execution

```bash
# 
Tests →  JSON Wire(legacy)/W3(New) protocol → Browser Driver → Actual Browser
                                                    ^               ^
                                                    |---------------|
```

This leads to heavier test runs.

For Cypress, it uses a Proxy that runs in an actual browser and is communicating via Websocket. It has amzing hot-reloading as it runs in actual browser with Websocket connection; but due to it has a couple of issues:

1. Since browser is heavy, it runs very slow when the number of test cases starts to increase.
    
2. Cypress was not extensible to Safari for the longest time. It started supporting Safari Webkit in 2022 by leveraging Playwright’ Webkit architecture.
    

Therefore Playwright is the best choice to use Snapshot for its versatility of supporting all browsers and providing mechanism to control network, can help in predictable and useful snapshot tests.

We can look into its doc on how to use [Visual comparisons](https://playwright.dev/docs/test-snapshots) in Playwright.

```javascript
import { test, expect } from '@playwright/test';

test('example test', async ({ page }) => {
  await page.goto('https://playwright.dev');
  await expect(page).toHaveScreenshot({ maxDiffPixels: 100 });
});
```

It is as simple as going to path and using the method `toHaveScreenshot` .

It provides 2 options:

1. `maxDiffPixels`: Allows for small variations to exist. It is so that the tests do not become too flaky.
    
2. `stylePath` : This allows for styles to be inserted to reduce the dynamic nature of screenshot. Could be a Layout style or differing scrollbar behaviors across browsers/OSes.
    

Even Vitest (which is a Jest API compatible testing framework by the Vite ecosystem) has an experimental [browser mode](https://vitest.dev/guide/browser/) (leveraging Playwright) that allows to write tests that run on an actual browser instead of environments such as JSDOM, HappyDOM.

It allows to use taking screenshots, but comparing and asserting it for new code is still not possible. See this blog by [Maya](https://mayashavin.com/articles/visual-testing-vitest-playwright) to read more on this topic.

### Example 3: Storybook

The latest version of Storybook (Version 9) has brought in different types of testing in the forefront while building components:  
1\. Interaction testing  
2\. Accessibility testing  
3\. Visual testing

Visual testing is the one we are most interested here. Since styles can be incorporated from an external source CSS, there is always a chance for it to get missed in unit jest-based test. Storybook along with Chromatic provides a cleaner way to manage this.  
It allows the concept of **Accepting** or **Denying** the change. You might want to accept it if it was a design system change, but you can also deny if it not intended and in fact a regression change.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1745733268309/fd2df294-616f-41a0-a870-c2f313a3e644.png align="center")

# Case against Snapshot tests

There are still some issues with using snapshot tests:  

1. Since snapshot tests scales to have a lot of images, it can lead to noise.
    
2. Sometimes there is also a psychological aspect to the validation of snapshot tests that make it less useful. A developer might have the scope of a particular component, but the failure might seem to be a false positive. So the developer can just update the snapshot and let it slide.
    

Having tools like [Chromatic](https://www.chromatic.com/), [Percy](https://percy.io/) and Applitools help alleviate this issue. They make it less overwhelming by providing an dashboard as well as review process to make sure the UI/UX are as intended.