---
id: message-extraction
title: Message Extraction
---

Now that you've declared some messages, it's time to extract them.

## Installation

```sh
npm i -D @formatjs/cli
```

## Extraction

```sh
formatjs extract "src/**/*.ts*" --out-file lang/en.json --id-interpolation-pattern '[sha512:contenthash:base64:6]'
```

:::caution ID Interpolation Pattern
Make sure this pattern matches `idInterpolationPattern` when you use `babel-plugin-react-intl` or `@formatjs/ts-transformer` in [Bundling with formatjs](https://formatjs.io/docs/guides/bundler-plugins) or you'll get a `MISSING_TRANSLATION` error.
:::

Given a file that has the following messages:

```tsx
import * as React from 'react';
import {FormattedMessage, useIntl, injectIntl} from 'react-intl';

class PasswordChangeWithIntl extends React.Component {
  render() {
    const {intl} = this.props;
    return (
      <li>
        <input
          placeholder={intl.formatMessage({
            defaultMessage: 'New Password',
            description: 'placeholder text',
          })}
        />
        <input
          placeholder={intl.formatMessage({
            id: 'explicit-id',
            defaultMessage: 'Confirm Password',
            description: 'placeholder text',
          })}
        />
      </li>
    );
  }
}

const PasswordChange = injectIntl(PasswordChangeWithIntl);

export function List(props) {
  const intl = useIntl();
  return (
    <section>
      <header>
        <FormattedMessage
          defaultMessage="Control Panel"
          description="title of control panel section"
        />
      </header>
      <ul>
        <li>
          <button>
            <FormattedMessage
              defaultMessage="Delete user {name}"
              description="Delete button"
              values={{
                name: props.name,
              }}
            />
          </button>
        </li>
        <PasswordChange />
      </ul>
    </section>
  );
}
```

running the above command will create a file called `lang/en.json`:

```json
{
  "hak27d": {
    "defaultMessage": "Control Panel",
    "description": "title of control panel section"
  },
  "haqsd": {
    "defaultMessage": "Delete user {name}",
    "description": "delete button"
  },
  "19hjs": {
    "defaultMessage": "New Password",
    "description": "placeholder text"
  },
  "explicit-id": {
    "defaultMessage": "Confirm Password",
    "description": "placeholder text"
  }
}
```

:::info Message ID
During extraction, we'll preserve explicit declared IDs and insert a hash as an ID for messages without. We recommend against explicit IDs since it can cause collision.
:::

## Automatic ID Generation

Since manual IDs are discouraged, we've provided a `babel` plugin and a `TypeScript` AST transformer that will automatically insert message IDs in your transpiled code. For more details please visit [Bundling with formatjs](https://formatjs.io/docs/guides/bundler-plugins).

## Translation Management System (TMS) Integration

The default format generated from `@formatjs/cli` might not work with the specific TMS/vendor you're working with. You can specify a custom formatter with `--format <formatFile>` that allows you to convert that format into something tailored to your TMS. For example:

If your vendor accepts the format like

```json
{
  "[id]": {
    "string": "[message]",
    "comment": "[description]"
  }
}
```

you can run

```sh
formatjs extract "src/**/*.ts*" --out-file lang/en.json --id-interpolation-pattern '[sha512:contenthash:base64:6]' --format formatter.js
```

where `formatter.js` is:

```js
exports.format = function (msgs) {
  const results = {};
  for (const [id, msg] of Object.entries(msgs)) {
    results[id] = {
      string: msg.defaultMessage,
      comment: msg.description,
    };
  }
  return results;
};
```

We also provide several [builtin formatters](../tooling/cli.md#builtin-formatters) so feel free to create PRs to add more.