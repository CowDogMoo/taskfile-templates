# 1Password CLI Tasks

Automated tasks for managing secrets, credentials, and items in 1Password using the 1Password CLI.

## Prerequisites

- [1Password CLI](https://developer.1password.com/docs/cli/get-started) (`op`) installed
- `jq` installed (required for `create-item` task)
- Authenticated 1Password session

## Quick Start

```bash
# Set up 1Password CLI and authenticate
task onepassword:setup-account OP_ACCOUNT=my-team.1password.com

# List available vaults
task onepassword:list-vaults

# Create a Login item
task onepassword:create-item \
  CATEGORY=Login \
  TITLE="GitHub" \
  VAULT_NAME=Personal \
  FIELDS='username=myuser,password=mypass' \
  URL=https://github.com
```

## Available Tasks

### Account Management

#### `setup-account`

Set up and authenticate with your 1Password account.

```bash
task onepassword:setup-account OP_ACCOUNT=my-team.1password.com
```

### Vault Management

#### `list-vaults`

List all vaults in your 1Password account.

```bash
task onepassword:list-vaults
```

#### `create-vault`

Create a new vault.

```bash
task onepassword:create-vault VAULT_NAME=Development
```

### Item Management

#### `create-item`

Create any type of 1Password item using templates. Supports all 23 item categories with proper field types.

**Parameters:**

- `CATEGORY` (required) - Item category (Login, Secure Note, Password, etc.)
- `TITLE` (required) - Item title/name
- `VAULT_NAME` (required) - Vault to store the item in
- `FIELDS` (optional) - Comma-separated field assignments
- `URL` (optional) - Primary URL (mainly for Login items)
- `TAGS` (optional) - Comma-separated tags

**Field Format:**

- `field=value` - Built-in field or custom text field
- `field=value:type` - Custom field with specific type

**Supported Field Types:**

- `text` - Text string (default)
- `password` - Concealed password field
- `email` - Email address
- `url` - Web address
- `date` - Date (format: YYYY-MM-DD)
- `monthYear` - Month and year (format: YYYYMM)
- `phone` - Phone number
- `otp` - One-time password (requires otpauth:// URI)

**Available Categories:**
Login, Secure Note, Password, Credit Card, Bank Account, Database, API Credential, SSH Key, Driver License, Identity, Membership, Outdoor License, Passport, Reward Program, Social Security Number, Software License, Email Account, Server, Wireless Router, Medical Record, Crypto Wallet, and Document.

**Examples:**

Create a Login item:

```bash
task onepassword:create-item \
  CATEGORY=Login \
  TITLE="GitHub Account" \
  VAULT_NAME=Personal \
  FIELDS='username=myusername,password=mysecurepassword' \
  URL=https://github.com
```

Create a Secure Note with multiple field types:

```bash
task onepassword:create-item \
  CATEGORY="Secure Note" \
  TITLE="Project Alpha Details" \
  VAULT_NAME=Work \
  FIELDS='project=AlphaProject,email=contact@example.com:email,website=https://docs.example.com:url,deadline=2025-12-31:date,phone=555-1234-5678:phone'
```

Create a Secure Note with concealed/password fields (for API credentials):

```bash
task onepassword:create-item \
  CATEGORY="Secure Note" \
  TITLE="API Credentials" \
  VAULT_NAME=Work \
  FIELDS='api_key=sk-1234567890:password,api_url=https://api.example.com:url,notes=Production API credentials'
```

Create a Password item (for API keys, tokens, etc.):

```bash
task onepassword:create-item \
  CATEGORY=Password \
  TITLE="Production API Key" \
  VAULT_NAME=Work \
  FIELDS='password=sk-1234567890abcdef,notesPlain=Production environment API key' \
  TAGS='api,production'
```

Create a Database item:

```bash
task onepassword:create-item \
  CATEGORY=Database \
  TITLE="Production DB" \
  VAULT_NAME=Work \
  FIELDS='hostname=db.example.com,database=myapp_prod,username=dbuser,password=dbpass123,port=5432'
```

Create an API Credential:

```bash
task onepassword:create-item \
  CATEGORY="API Credential" \
  TITLE="Stripe API" \
  VAULT_NAME=Work \
  FIELDS='username=stripe_user,credential=sk_live_abc123,api_endpoint=https://api.stripe.com:url' \
  TAGS='payment,api'
```

#### `delete-item`

Delete an item from 1Password (requires confirmation).

```bash
task onepassword:delete-item \
  ITEM_TITLE="Old Item" \
  VAULT_NAME=Personal
```

### Secret Retrieval

#### `list-secrets`

List secrets in a vault, optionally filtered by category.

```bash
# List all secrets in a vault
task onepassword:list-secrets VAULT_NAME=Personal

# List only Login items
task onepassword:list-secrets VAULT_NAME=Personal CATEGORY=Login
```

#### `get-secret`

Retrieve a specific secret from 1Password.

```bash
# Get a secret by name
task onepassword:get-secret SECRET_NAME="GitHub Token" VAULT=Personal

# Get a specific field
task onepassword:get-secret SECRET_NAME="GitHub Token" VAULT=Personal FIELD=password

# Save to file
task onepassword:get-secret SECRET_NAME="SSH Key" VAULT=Personal OUTPUT_FILE=~/.ssh/id_rsa
```

## Field Types Reference

### Built-in Fields

These are predefined in each item template:

- **Login**: `username`, `password`, `notesPlain`
- **Password**: `password`, `notesPlain`
- **Secure Note**: `notesPlain`
- **Database**: `hostname`, `database`, `username`, `password`, `port`

Built-in fields don't require a type specification - just use `field=value`.

### Custom Fields

Any field not in the template becomes a custom field. Specify the type with `:type` suffix:

- `email=user@example.com:email`
- `website=https://example.com:url`
- `birthday=1990-01-15:date`
- `secret_key=abc123:password`

Without a type specification, custom fields default to `text` type.

## Tips

1. **Built-in fields** - Check available built-in fields with `op item template get <category>`
2. **Tags for organization** - Use tags to categorize items: `TAGS='production,api,critical'`
3. **URL field for Logins** - Always set the URL parameter for Login items to enable autofill
4. **Date format** - Use YYYY-MM-DD format for date fields
5. **Debug mode** - Use `DEBUG=1` to see the generated JSON template: `DEBUG=1 task onepassword:create-item ...`

## Troubleshooting

### "jq command not found"

Install jq:

```bash
# macOS
brew install jq

# Ubuntu/Debian
sudo apt-get install jq
```

### "Not signed in to 1Password"

Run the setup task:

```bash
task onepassword:setup-account OP_ACCOUNT=your-account.1password.com
```

### "Failed to fetch template for category"

List available categories:

```bash
op item template list
```

### Field not appearing correctly

Check the template structure:

```bash
op item template get "Login"
```

## Resources

- [1Password CLI Documentation](https://developer.1password.com/docs/cli)
- [Item Templates Reference](https://developer.1password.com/docs/cli/item-template-json)
- [Field Types Reference](https://developer.1password.com/docs/cli/item-fields)
