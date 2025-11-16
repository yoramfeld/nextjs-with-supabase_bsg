# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Next.js 15+ application with Supabase authentication using the App Router. Built with TypeScript, Tailwind CSS, and shadcn/ui components (New York style).

## Development Commands

```bash
# Start development server with Turbopack
npm run dev

# Build for production
npm run build

# Run production build
npm start

# Lint code
npm run lint
```

## Architecture

### Supabase Client Patterns

This project uses three distinct Supabase client initializations for different contexts:

1. **Browser Client** (`lib/supabase/client.ts`): For Client Components
   - Uses `createBrowserClient` from `@supabase/ssr`
   - Import: `import { createClient } from "@/lib/supabase/client"`

2. **Server Client** (`lib/supabase/server.ts`): For Server Components, Route Handlers, and Server Actions
   - Uses `createServerClient` with cookie handling via Next.js `cookies()`
   - **CRITICAL**: Always create a new client instance within each function - never use global variables (important for Fluid compute)
   - Import: `import { createClient } from "@/lib/supabase/server"`

3. **Middleware Client** (`lib/supabase/middleware.ts`): For session refresh
   - Handles session token refresh on each request
   - Redirects unauthenticated users trying to access protected routes
   - Returns the response object with updated cookies

### Authentication Flow

- Cookie-based sessions using `@supabase/ssr` package
- Middleware (`middleware.ts`) runs on all routes except static files and images
- Protected routes redirect to `/auth/login` if user is not authenticated
- Auth routes: `/auth/login`, `/auth/sign-up`, `/auth/forgot-password`, `/auth/update-password`
- The `/protected` route demonstrates authenticated-only pages

### Middleware Behavior

The middleware:
- Calls `supabase.auth.getClaims()` to refresh the session
- **IMPORTANT**: Never run code between `createServerClient` and `supabase.auth.getClaims()` as it can cause random logouts
- Redirects unauthenticated users to `/auth/login` for non-public routes
- Must return the `supabaseResponse` object with cookies intact

### Path Aliases

TypeScript path aliases are configured in `tsconfig.json`:
- `@/*` maps to the root directory
- Example: `import { createClient } from "@/lib/supabase/server"`

### Component Organization

- `components/ui/` - shadcn/ui components (badge, button, card, checkbox, dropdown-menu, input, label)
- `components/` - Application components (auth forms, hero, theme switcher, etc.)
- `components/tutorial/` - Tutorial step components for onboarding
- `app/` - Next.js App Router pages and layouts
- `app/auth/` - Authentication pages and route handlers
- `app/protected/` - Example protected routes

### Styling

- Tailwind CSS with custom configuration
- shadcn/ui components with "New York" style
- Theme switching support (light/dark mode via `next-themes`)
- CSS variables for theming defined in `app/globals.css`

## Environment Variables

Required variables in `.env.local`:
```env
NEXT_PUBLIC_SUPABASE_URL=your-project-url
NEXT_PUBLIC_SUPABASE_PUBLISHABLE_KEY=your-publishable-or-anon-key
```

Get these from: https://app.supabase.com/project/_/settings/api

**Note**: Both `NEXT_PUBLIC_*` variables are intentionally exposed to the browser. The publishable/anon key is safe for client-side use as it respects Row Level Security (RLS) policies.

## Adding New shadcn/ui Components

```bash
npx shadcn@latest add [component-name]
```

The `components.json` file configures shadcn/ui with New York style. If you want to change styles, delete `components.json` and re-initialize shadcn/ui.

## Protected Routes

To create a new protected route:
1. Create your page in `app/your-route/page.tsx`
2. Use the server Supabase client to check authentication:
```typescript
import { createClient } from "@/lib/supabase/server";
import { redirect } from "next/navigation";

export default async function YourPage() {
  const supabase = await createClient();
  const { data, error } = await supabase.auth.getClaims();

  if (error || !data?.claims) {
    redirect("/auth/login");
  }

  // Your protected page content
}
```

The middleware will also catch unauthenticated access attempts and redirect to `/auth/login`.

## Important Notes

- **Turbopack** is enabled by default in dev mode (`--turbopack` flag)
- React 19 is used in this project
- Authentication uses the Supabase UI Library for password-based auth
- The starter comes with pre-built auth forms - customize them in `components/` directory
- For local Supabase development, see: https://supabase.com/docs/guides/getting-started/local-development

## Testing Authentication Locally

1. Ensure `.env.local` is configured with your Supabase credentials
2. Run `npm run dev`
3. Visit http://localhost:3000
4. Sign up for a new account via `/auth/sign-up`
5. Check email for confirmation (or disable email confirmation in Supabase dashboard for testing)
6. Access `/protected` to test authenticated routes
