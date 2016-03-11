# A silly little tutorial on composing React components

For those that are curious

## The Problem

Let's say you have some view that looks like so:

```js
// ./SomeView.js

import React from 'react';
import IconLabelValueRow from './IconLabelValueRow';
import EditLabelRow from './EditLabelRow';

function SomeView(props) {
  return (
    <div className='some-view'>
      <div className='space-top-2 space-4'>
        <IconLabelValueRow
          iconClass='icon-beach icon-doorman'
          label='Some Label'
          value='Some Value'
        />
      </div>
      <div className='space-top-8 space-4'>
        <EditValueRow label='Something' />
      </div>
    </div>
  );
}
```

We see a common pattern of wrapping our components in div with some positioning classes (they apply top & bottom margins).

```js
<div className='space-top-2 space-4'>
  // <IconLabelValueRow />
</div>
<div className='space-top-8 space-4'>
  // <EditValueRow />
</div>
```

## One way to solve it

One solution would be add pass a spacing value through props and add the value to the className on `IconLabelValueRow`.

```js
// ./IconLabelValueRow.js

import React from 'react';
import cx from 'classnames';

function IconLabelValueRow(props) {
  const {
    iconClass, label, value,
    spaceTop, spaceBottom
  } = props;

  const classes = cx({
    [`space-top-${spaceTop}`]: !!spaceTop,
    [`space-${spaceBottom}`]: !!spaceBottom,
  }, 'flexbox flexbox--row flexbox--align-center');

  return (
    <div className={classes}>
      <div><i className={`icon ${iconClass}`} /></div>
      <div>{label}</div>
      <div>{value}</div>
    </div>
  );
}
```

Then `SomeView` would look like this:

```js
// ./SomeView.js

function SomeView(props) {
  return (
    <div className='some-view'>
      <IconLabelValueRow
        iconClass='icon-beach icon-doorman'
        label='Some Label'
        value='Some Value'
        spaceTop={2}
        spaceBottom={8}
      />
      <div className='space-top-8 space-4'>
        <EditValueRow label='Something' />
      </div>
    </div>
  );
}
```

## Another problem

But this doesn't solve the problem for `EditValueRow`. We would have to copy and paste the `spaceTop` and `spaceBottom` props preamble over to the other component like so:

```js
// ./EditLabelRow.js

import React from 'react';
import cx from 'classnames';

export default function EditLabelRow(props) {
  const { label, spaceTop, spaceBottom } = props;

  const classes = cx({
    [`space-top-${spaceTop}`]: !!spaceTop,
    [`space-${spaceBottom}`]: !!spaceBottom,
  }, 'flexbox');

  return (
    <div className={classes}>
      <input placeholder={label} />
    </div>
  );
}
```

So now `SomeView` looks like this:

```js
// ./SomeView.js

function SomeView(props) {
  return (
    <div className='some-view'>
      <IconLabelValueRow
        iconClass='icon-beach icon-doorman'
        label='Some Label'
        value='Some Value'
        spaceTop={2}
        spaceBottom={8}
      />
      <EditValueRow
        label='Something'
        spaceTop={8}
        spaceBottom={4}
      />
    </div>
  );
}
```

But copy + paste makes us sad so how do you decorate your component in a composable way?

## A better solution

Enter higher-order functions. A higher-order function takes a component and in this example returns a state-less function.

```js
// ./enhancers/Spacing.js

import React from 'react';
import cx from 'classnames';

export default function Spacing(Component) {
  return (props) => {
    const { spaceTop, spaceBottom } = props;
    const classes = cx({
      [`space-top-${spaceTop}`]: !!spaceTop,
      [`space-${spaceBottom}`]: !!spaceBottom,
    });

    return (
      <div className={classes}>
        <Component {...props} />
      </div>
    );
  }
}
```

And now we can use this in `IconLabelValueRow`.

```js
// ./IconLabelValueRow.js

import React from 'react';
import Spacing from './enhancers/Spacing';

function IconLabelValueRow(props) {
  const { iconClass, label, value } = props;
  return (
    <div className='flexbox flexbox--row flexbox--align-center'>
      <div><i className={`icon ${iconClass}`} /></div>
      <div>{label}</div>
      <div>{value}</div>
    </div>
  );
}

/**
 * only difference is we export a composed component
 */
export default Spacing(IconLabelValueRow);
```

and in `EditLabelRow`:

```js
// ./EditLabelRow.js

import React from 'react';
import Spacing from './enhancers/Spacing';

function EditLabelRow(props) {
  const { label } = props;
  return (
    <div>
      <input placeholder={label} />
    </div>
  );
}

export default Spacing(EditLabelRow);
```

And now we live in a happy world.

```js
// ./SomeView.js

import IconLabelValueRow from './IconLabelValueRow';
import EditLabelRow from './EditLabelRow';

function SomeView(props) {
  return (
    <div className='some-view'>
      <IconLabelValueRow
        iconClass='icon-beach icon-doorman'
        label='Some Label'
        value='Some Value'
        spaceTop={2}
        spaceBottom={4}
      />
      <EditValueRow
        label='Something'
        spaceTop={8}
        spaceBottom={4}
      />
    </div>
  );
}
```
