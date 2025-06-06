---
sidebar_position: 13
---

# Share A Lock (v1)

This endpoint allows operations to be performed on a lock, such as lock, unlock, add/remove user. Requests to this endpoint must be signed and formed as a JSON Web Token (JWT).

### HTTP Request

`POST https://api.doordeck.com/device/LOCK_ID/execute`

:::tip[Success]
If a request expires within the next 60 seconds, a 204 is returned upon success, if a request expires in more than 60 seconds, a 202 is returned to indicate the request has been queued for the device.
:::

### Request Parameters

import SecureOperationParameters from './partials/_secure-operation-params.md';

<SecureOperationParameters name="Secure operation parameters" />

The operation object definition is as follows

| Parameter | Required | Description                                                                               |
|-----------|----------|-------------------------------------------------------------------------------------------|
| type      | true     | Must be `ADD_USER`                                                                        |
| user      | true     | ID of user to add                                                                         |
| publicKey | true     | Public key of user to add                                                                 |
| role      | false    | Should be either ADMIN or USER                                                            |
| start     | false    | Unix timestamp of when the user should be active from, null or unset to start immediately |
| end       | false    | Unix timestamp of when the user should expire, null or unset for never expires            |

After the request has been signed with user's ephemeral key, it should be sent to the execute endpoint.

### Example

import Tabs from '@theme/Tabs';
import TabItem from '@theme/TabItem';
import CodeBlock from '@theme/CodeBlock';
import Setup from '!!raw-loader!./partials/shell/_secure-operation-setup.sh';
import Execute from '!!raw-loader!./partials/shell/_secure-operation-execute.sh';

<Tabs>
<TabItem value="request" label="Request">

<CodeBlock language="shell" title="CURL">

{`${Setup}

BODY=$(
  jq -n \\
    --arg iss "$USER_ID" \\
    --arg sub "$DEVICE_ID" \\
    --arg jti "$(uuidgen)" \\
    --arg publicKey "INVITEE_PUBLIC_KEY" \\
    --arg user "11111111-1111-1111-1111-111111111111" \\
    --arg role "USER" \\
    --argjson nbf "$(date +%s)" \\
    --argjson iat "$(date +%s)" \\
    --argjson exp "$(($(date +%s) + 60))" \\
    --argjson start "$USER_START_TIME" \\
    --argjson end "$USER_END_TIME" \\
    '{
      iss: $iss,
      sub: $sub,
      jti: $jti,
      nbf: $nbf,
      iat: $iat,
      exp: $exp,
      operation: {
        type: "ADD_USER",
        publicKey: $publicKey,
        user: $user,
        role: $role,
        start: $start,
        end: $end
      }
    }'
)

${Execute}`}

</CodeBlock>

:::info[Remember]
* Replace `TOKEN` with your access token.
* Replace `00000000-0000-0000-0000-000000000000` with the lock's ID.
* Replace `INVITEE_PUBLIC_KEY` with the invitee's public key.
* Replace `11111111-1111-1111-1111-111111111111` with the invitee's user ID.
* Replace `USER_START_TIME` and `USER_END_TIME` with Unix timestamps of when the user should be activated from and until, use null for indefinite.
:::

</TabItem>
<TabItem value="response" label="Response">

```json showLineNumbers title="JSON"
{}
```

</TabItem>
</Tabs>
