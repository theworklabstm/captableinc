## Basics

### Subject
The subject or subject type which you want to check user action on. Usually this is a business (or domain) entity (e.g., billing, roles, members). subjects can be added in the `subjects.ts` file


### Action
explains what users are able to do in the app. User actions are typically verbs determined by how the business operates. Often, these actions will include words like create, read, update, and delete. actions can be added in the `actions.ts` file

## Usage

### tRPC procedure

```javascript

import { withAccessControl } from "@/trpc/api/trpc";

export const withAccessControlProcedure = withAccessControl
  .input(inputSchema)
  .meta({
    policies: {
      members: { allow: ["create"] },
    },
  })
  .mutation(({ ctx, input }) => {
    const { membership } = ctx;
    return { success: true };
  });
``` 

### Client Components

```javascript

"use client"

import { Allow } from "@/components/rbac/allow";

function ClientComponent() {
  return (
    <div>
      <Allow action="delete" subject="roles">
        Allowed
      </Allow>

      {/* or using render props */}

      <Allow action="delete" subject="roles">
        {(allow) => (allow ? "allowed" : "disallowed")}
      </Allow>
    </div>
  );
}

``` 

### Server Components

```javascript

"use server";

import { serverAccessControl } from "@/lib/rbac/access-control";

const fetchDataFromServer = async () => {
  return { data: [] };
};

async function ServerComponent() {
  const { allow } = await serverAccessControl();

  const canRead = !!allow(true, ["billing", "read"]);
  const data = await allow(fetchDataFromServer(), ["billing", "read"]);

  return (
    <div>
      {canRead ? "can read" : "cannot read"}

      {data ? data.data : null}
    </div>
  );
}


``` 


