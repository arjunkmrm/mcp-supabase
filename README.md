# Supabase MCP Server

A Model Context Protocol (MCP) server that provides comprehensive tools for interacting with Supabase databases, storage, and edge functions. This server enables seamless integration between Supabase services and MCP-compatible applications.


## Overview

The Supabase MCP server acts as a bridge between MCP clients and Supabase's suite of services, providing:

- Database operations with rich querying capabilities
- Storage management for files and assets
- Edge function invocation
- Project and organization management
- User authentication and management
- Role-based access control


## MCP Integration

Add the server to your MCP settings:

```json
{
  "mcpServers": {
    "supabase": {
      "command": "npx",
      "args": [
        "-y",
        "mcp-supabase"
        ],
      "env": {
        "SUPABASE_URL": "your_project_url",
        "SUPABASE_KEY": "your_service_role_key",
        "SUPABASE_ACCESS_TOKEN": "your_access_token"
      },
      "config": "path/to/config.json"  // Optional: path to configuration file
    }
  }
}
```

### Supabase Configuration
```json
{
  "supabase": {
    "project": {
      "url": "your_project_url",
      "key": "your_service_role_key",
      "accessToken": "your_access_token"
    },
    "storage": {
      "defaultBucket": "public",           // Default storage bucket
      "maxFileSize": 52428800,            // Max file size in bytes (50MB)
      "allowedMimeTypes": [               // Allowed file types
        "image/*",
        "application/pdf",
        "text/*"
      ]
    },
    "database": {
      "maxConnections": 10,               // Max DB connections
      "timeout": 30000,                   // Query timeout in ms
      "ssl": true                         // SSL connection
    },
    "auth": {
      "autoConfirmUsers": false,          // Auto-confirm new users
      "disableSignup": false,             // Disable public signups
      "jwt": {
        "expiresIn": "1h",               // Token expiration
        "algorithm": "HS256"              // JWT algorithm
      }
    }
  }
}
```

### Logging Configuration
```json
{
  "logging": {
    "level": "info",                      // Log level
    "format": "json",                     // Log format
    "outputs": ["console", "file"],       // Output destinations
    "file": {
      "path": "logs/server.log",          // Log file path
      "maxSize": "10m",                   // Max file size
      "maxFiles": 5                       // Max number of files
    }
  }
}
```

### Security Configuration
```json
{
  "security": {
    "cors": {
      "enabled": true,
      "origins": ["*"],
      "methods": ["GET", "POST", "PUT", "DELETE", "OPTIONS"],
      "allowedHeaders": ["Content-Type", "Authorization"]
    },
    "rateLimit": {
      "enabled": true,
      "windowMs": 900000,                 // 15 minutes
      "max": 100                          // Max requests per window
    }
  }
}
```

### Monitoring Configuration
```json
{
  "monitoring": {
    "enabled": true,
    "metrics": {
      "collect": true,
      "interval": 60000                   // Collection interval in ms
    },
    "health": {
      "enabled": true,
      "path": "/health"                   // Health check endpoint
    }
  }
}
```

See `config.json.example` for a complete example configuration file.

## Available Tools

### Database Operations

#### create_record
Create a new record in a table with support for returning specific fields.

```typescript
{
  table: string;
  data: Record<string, any>;
  returning?: string[];
}
```

Example:
```typescript
{
  table: "users",
  data: {
    name: "John Doe",
    email: "john@example.com"
  },
  returning: ["id", "created_at"]
}
```

#### read_records
Read records with advanced filtering, joins, and field selection.

```typescript
{
  table: string;
  select?: string[];
  filter?: Record<string, any>;
  joins?: Array<{
    type?: 'inner' | 'left' | 'right' | 'full';
    table: string;
    on: string;
  }>;
}
```

Example:
```typescript
{
  table: "posts",
  select: ["id", "title", "user.name"],
  filter: { published: true },
  joins: [{
    type: "left",
    table: "users",
    on: "posts.user_id=users.id"
  }]
}
```

#### update_record
Update records with filtering and returning capabilities.

```typescript
{
  table: string;
  data: Record<string, any>;
  filter?: Record<string, any>;
  returning?: string[];
}
```

Example:
```typescript
{
  table: "users",
  data: { status: "active" },
  filter: { email: "john@example.com" },
  returning: ["id", "status", "updated_at"]
}
```

#### delete_record
Delete records with filtering and returning capabilities.

```typescript
{
  table: string;
  filter?: Record<string, any>;
  returning?: string[];
}
```

Example:
```typescript
{
  table: "posts",
  filter: { status: "draft" },
  returning: ["id", "title"]
}
```

### Storage Operations

#### upload_file
Upload files to Supabase Storage with configurable options.

```typescript
{
  bucket: string;
  path: string;
  file: File | Blob;
  options?: {
    cacheControl?: string;
    contentType?: string;
    upsert?: boolean;
  };
}
```

Example:
```typescript
{
  bucket: "avatars",
  path: "users/123/profile.jpg",
  file: imageBlob,
  options: {
    contentType: "image/jpeg",
    upsert: true
  }
}
```

#### download_file
Download files from Supabase Storage.

```typescript
{
  bucket: string;
  path: string;
}
```

Example:
```typescript
{
  bucket: "documents",
  path: "reports/annual-2023.pdf"
}
```

### Edge Functions

#### invoke_function
Invoke Supabase Edge Functions with parameters and custom options.

```typescript
{
  function: string;
  params?: Record<string, any>;
  options?: {
    headers?: Record<string, string>;
    responseType?: 'json' | 'text' | 'arraybuffer';
  };
}
```

Example:
```typescript
{
  function: "process-image",
  params: {
    url: "https://example.com/image.jpg",
    width: 800
  },
  options: {
    responseType: "json"
  }
}
```

### User Management

#### list_users
List users with pagination support.

```typescript
{
  page?: number;
  per_page?: number;
}
```

#### create_user
Create a new user with metadata.

```typescript
{
  email: string;
  password: string;
  data?: Record<string, any>;
}
```

#### update_user
Update user details.

```typescript
{
  user_id: string;
  email?: string;
  password?: string;
  data?: Record<string, any>;
}
```

#### delete_user
Delete a user.

```typescript
{
  user_id: string;
}
```

#### assign_user_role
Assign a role to a user.

```typescript
{
  user_id: string;
  role: string;
}
```

#### remove_user_role
Remove a role from a user.

```typescript
{
  user_id: string;
  role: string;
}
```

## Error Handling

The server provides detailed error messages for common scenarios:

- Invalid parameters
- Authentication failures
- Permission issues
- Rate limiting
- Network errors
- Database constraints

Errors are returned in a standardized format:

```typescript
{
  code: ErrorCode;
  message: string;
  details?: any;
}
```

## Development

### Running Tests
```bash
npm test
```

### Building
```bash
npm run build
```

### Linting
```bash
npm run lint
```

## Contributing

1. Fork the repository
2. Create a feature branch
3. Commit your changes
4. Push to the branch
5. Create a Pull Request

## License

MIT License - see LICENSE for details
