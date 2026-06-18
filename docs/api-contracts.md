# API Contracts

## General API Rules

- Use HTTPS only
- Version APIs with `/v1`
- Use JSON request and response bodies
- Include `requestId` for tracing
- Use cursor pagination for changing datasets
- Use idempotency keys for write operations
- Enforce workspace authorization on every workspace-scoped request

Common headers:

```http
Authorization: Bearer <access_token>
X-Request-Id: <uuid>
Idempotency-Key: <uuid>
```

Common error response:

```json
{
  "error": {
    "code": "permission_denied",
    "message": "You do not have access to this workspace.",
    "requestId": "req_123"
  }
}
```

## Authentication

### Register

`POST /v1/auth/register`

```json
{
  "email": "user@example.com",
  "password": "string",
  "displayName": "Alex"
}
```

Response:

```json
{
  "user": {
    "id": "usr_123",
    "email": "user@example.com",
    "displayName": "Alex"
  },
  "accessToken": "jwt",
  "refreshToken": "opaque_refresh_token"
}
```

### Login

`POST /v1/auth/login`

```json
{
  "email": "user@example.com",
  "password": "string",
  "deviceId": "dev_123"
}
```

### Refresh Token

`POST /v1/auth/refresh`

```json
{
  "refreshToken": "opaque_refresh_token"
}
```

### Logout

`POST /v1/auth/logout`

```json
{
  "refreshToken": "opaque_refresh_token",
  "allDevices": false
}
```

## User Profile

### Current User

`GET /v1/me`

Response:

```json
{
  "id": "usr_123",
  "email": "user@example.com",
  "displayName": "Alex",
  "createdAt": "2026-06-18T09:00:00Z"
}
```

## Workspaces

### List Workspaces

`GET /v1/workspaces`

Response:

```json
{
  "items": [
    {
      "id": "wks_123",
      "name": "Personal",
      "type": "personal",
      "role": "owner"
    }
  ]
}
```

### Create Workspace

`POST /v1/workspaces`

```json
{
  "name": "Team Alpha",
  "type": "team"
}
```

### Invite Member

`POST /v1/workspaces/{workspaceId}/invites`

```json
{
  "email": "member@example.com",
  "role": "editor"
}
```

### Update Member Role

`PATCH /v1/workspaces/{workspaceId}/members/{userId}`

```json
{
  "role": "viewer"
}
```

## Settings

### Get Settings

`GET /v1/workspaces/{workspaceId}/settings`

### Update Settings

`PATCH /v1/workspaces/{workspaceId}/settings`

```json
{
  "timezone": "Europe/Zurich",
  "locale": "de-CH",
  "featureFlags": {
    "newDashboard": true
  }
}
```

## Sync

### Pull Changes

`GET /v1/workspaces/{workspaceId}/sync/changes?cursor=<cursor>&limit=500`

Response:

```json
{
  "items": [
    {
      "changeId": "chg_123",
      "entityType": "task",
      "entityId": "tsk_123",
      "operation": "upsert",
      "version": 4,
      "changedAt": "2026-06-18T09:00:00Z",
      "data": {}
    }
  ],
  "nextCursor": "cur_456",
  "hasMore": false
}
```

### Push Operations

`POST /v1/workspaces/{workspaceId}/sync/operations`

```json
{
  "clientId": "dev_123",
  "operations": [
    {
      "operationId": "op_123",
      "entityType": "task",
      "entityId": "tsk_123",
      "operation": "upsert",
      "baseVersion": 3,
      "data": {}
    }
  ]
}
```

Response:

```json
{
  "accepted": [
    {
      "operationId": "op_123",
      "entityId": "tsk_123",
      "version": 4
    }
  ],
  "conflicts": []
}
```

## Generic Feature Resources

Use this only for generic modules. Complex domains should define specific endpoints.

### List Resources

`GET /v1/workspaces/{workspaceId}/resources/{resourceType}?cursor=<cursor>`

### Create Resource

`POST /v1/workspaces/{workspaceId}/resources/{resourceType}`

### Update Resource

`PATCH /v1/workspaces/{workspaceId}/resources/{resourceType}/{resourceId}`

### Delete Resource

`DELETE /v1/workspaces/{workspaceId}/resources/{resourceType}/{resourceId}`

## Audit Events

### Create Audit Event

`POST /v1/audit/events`

```json
{
  "workspaceId": "wks_123",
  "type": "security.permission_denied",
  "entityType": "task",
  "entityId": "tsk_123",
  "metadata": {
    "permission": "task.update"
  }
}
```

## Client Module Contracts

### Auth Client

```ts
interface AuthClient {
  getSession(): Promise<AuthSession | null>;
  login(email: string, password: string): Promise<AuthSession>;
  register(input: RegisterInput): Promise<AuthSession>;
  refresh(): Promise<AuthSession>;
  logout(options?: { allDevices?: boolean }): Promise<void>;
}
```

### Workspace Client

```ts
interface WorkspaceClient {
  list(): Promise<Workspace[]>;
  create(input: CreateWorkspaceInput): Promise<Workspace>;
  switch(workspaceId: string): Promise<void>;
  invite(workspaceId: string, input: InviteInput): Promise<Invite>;
}
```

### Local Repository

```ts
interface LocalRepository<T> {
  get(id: string): Promise<T | null>;
  list(query?: LocalQuery): Promise<T[]>;
  save(entity: T): Promise<T>;
  delete(id: string): Promise<void>;
  observe(query?: LocalQuery): Observable<T[]>;
}
```

### Sync Engine

```ts
interface SyncEngine {
  status(): SyncStatus;
  syncNow(workspaceId: string): Promise<SyncResult>;
  enqueue(operation: SyncOperation): Promise<void>;
  resolveConflict(input: ConflictResolution): Promise<void>;
}
```
