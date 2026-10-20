# Nueltrades
nuel-trades-ai/
├── package.json
├── tsconfig.json
├── next.config.mjs
├── tailwind.config.ts
├── postcss.config.mjs
├── .env.example
├── prisma/
│   └── schema.prisma
├── src/
│   ├── lib/
│   │   ├── db.ts
│   │   ├── auth.ts
│   │   ├── validation.ts
│   │   ├── analytics.ts
│   │   ├── storage.ts
│   │   ├── ai.ts
│   │   └── api.ts
│   ├── middleware.ts
│   ├── app/
│   │   ├── layout.tsx
│   │   ├── globals.css
│   │   ├── page.tsx
│   │   ├── login/page.tsx
│   │   ├── register/page.tsx
│   │   ├── dashboard/page.tsx
│   │   ├── accounts/page.tsx
│   │   ├── trades/page.tsx
│   │   ├── trades/new/page.tsx
│   │   ├── trades/[id]/page.tsx
│   │   ├── import/page.tsx
│   │   └── api/
│   │       ├── auth/[...nextauth]/route.ts
│   │       ├── auth/register/route.ts
│   │       ├── accounts/route.ts
│   │       ├── accounts/[id]/route.ts
│   │       ├── trades/route.ts
│   │       ├── trades/[id]/route.ts
│   │       ├── analytics/route.ts
│   │       ├── import/csv/route.ts
│   │       ├── ai/analyze-trade/route.ts
│   │       └── uploads/route.ts
│   └── components/
│       ├── Nav.tsx
│       ├── MetricCard.tsx
│       ├── EquityCurve.tsx
│       └── TradeForm.tsx
{
  "name": "nuel-trades-ai",
  "version": "0.1.0",
  "private": true,
  "scripts": {
    "dev": "next dev",
    "build": "prisma generate && next build",
    "start": "next start",
    "lint": "next lint",
    "db:push": "prisma db push",
    "db:migrate": "prisma migrate dev",
    "db:studio": "prisma studio",
    "test": "vitest run"
  },
  "dependencies": {
    "@auth/prisma-adapter": "^2.7.4",
    "@prisma/client": "^5.22.0",
    "@aws-sdk/client-s3": "^3.685.0",
    "@aws-sdk/s3-request-presigner": "^3.685.0",
    "@anthropic-ai/sdk": "^0.32.1",
    "bcryptjs": "^2.4.3",
    "next": "14.2.15",
    "next-auth": "5.0.0-beta.25",
    "react": "^18.3.1",
    "react-dom": "^18.3.1",
    "recharts": "^2.13.3",
    "zod": "^3.23.8"
  },
  "devDependencies": {
    "@types/bcryptjs": "^2.4.6",
    "@types/node": "^22.9.0",
    "@types/react": "^18.3.12",
    "@types/react-dom": "^18.3.1",
    "autoprefixer": "^10.4.20",
    "postcss": "^8.4.49",
    "prisma": "^5.22.0",
    "tailwindcss": "^3.4.14",
    "typescript": "^5.6.3",
    "vitest": "^2.1.4"
  }
}
{
  "compilerOptions": {
    "target": "ES2022",
    "lib": ["dom", "dom.iterable", "esnext"],
    "allowJs": false,
    "skipLibCheck": true,
    "strict": true,
    "noEmit": true,
    "esModuleInterop": true,
    "module": "esnext",
    "moduleResolution": "bundler",
    "resolveJsonModule": true,
    "isolatedModules": true,
    "jsx": "preserve",
    "incremental": true,
    "plugins": [{ "name": "next" }],
    "paths": { "@/*": ["./src/*"] }
  },
  "include": ["next-env.d.ts", "**/*.ts", "**/*.tsx", ".next/types/**/*.ts"],
  "exclude": ["node_modules"]
}
/** @type {import('next').NextConfig} */
const nextConfig = {
  experimental: { serverActions: { bodySizeLimit: "10mb" } },
};
export default nextConfig;
// tailwind.config.ts
import type { Config } from "tailwindcss";
export default {
  content: ["./src/**/*.{ts,tsx}"],
  theme: {
    extend: {
      colors: {
        bg: "#0a0a0b",
        panel: "#131316",
        border: "#26262b",
        muted: "#8b8b93",
        pos: "#22c55e",
        neg: "#ef4444",
        warn: "#f59e0b",
      },
    },
  },
  plugins: [],
} satisfies Config;
// postcss.config.mjs
export default { plugins: { tailwindcss: {}, autoprefixer: {} } };
/* src/app/globals.css */
@tailwind base;
@tailwind components;
@tailwind utilities;

html, body { background: #0a0a0b; color: #f5f5f7; }
body { font-family: ui-sans-serif, system-ui, -apple-system, "Segoe UI", Roboto, sans-serif; }
input, select, textarea {
  background: #0a0a0b; border: 1px solid #26262b; color: #f5f5f7;
  border-radius: 6px; padding: 8px 10px; outline: none;
}
input:focus, select:focus, textarea:focus { border-color: #22c55e; }
button { cursor: pointer; }
DATABASE_URL="postgresql://postgres:postgres@localhost:5432/nueltrades?schema=public"
NEXTAUTH_URL="http://localhost:3000"
NEXTAUTH_SECRET="generate-with-openssl-rand-base64-32"

# Optional Google sign-in
GOOGLE_CLIENT_ID=""
GOOGLE_CLIENT_SECRET=""

# AI provider (server-side only, never exposed to client)
AI_PROVIDER="anthropic"           # "anthropic" | "openai" | ""
ANTHROPIC_API_KEY=""
OPENAI_API_KEY=""
AI_MODEL="claude-3-5-sonnet-20241022"

# S3-compatible storage (MinIO / AWS / R2)
S3_ENDPOINT="http://localhost:9000"
S3_REGION="us-east-1"
S3_BUCKET="nuel-uploads"
S3_ACCESS_KEY="minioadmin"
S3_SECRET_KEY="minioadmin"
S3_FORCE_PATH_STYLE="true"
generator client { provider = "prisma-client-js" }
datasource db { provider = "postgresql"; url = env("DATABASE_URL") }

enum AccountKind     { LIVE DEMO PROP }
enum TradeSide       { BUY SELL }
enum TradeStatus     { OPEN CLOSED }
enum ImportSource    { MANUAL CSV_STATEMENT BROKER_SYNC }
enum JournalCategory { REAL DEMO BACKTEST PROP STRATEGY CUSTOM }

model User {
  id            String   @id @default(cuid())
  email         String   @unique
  passwordHash  String?
  name          String?
  image         String?
  emailVerified DateTime?
  createdAt     DateTime @default(now())
  updatedAt     DateTime @updatedAt

  accounts      TradingAccount[]
  trades        Trade[]
  journals      Journal[]
  strategies    Strategy[]
  aiAnalyses    AIAnalysis[]
  screenshots   Screenshot[]
  auditEvents   AuditEvent[]
}

model TradingAccount {
  id           String      @id @default(cuid())
  userId       String
  name         String
  broker       String?
  currency     String      @default("USD")
  kind         AccountKind
  initialBalance Decimal   @db.Decimal(18, 2) @default(0)
  isArchived   Boolean     @default(false)
  createdAt    DateTime    @default(now())
  updatedAt    DateTime    @updatedAt

  user         User        @relation(fields: [userId], references: [id], onDelete: Cascade)
  trades       Trade[]
  transactions Transaction[]

  @@index([userId])
}

model Transaction {
  id        String   @id @default(cuid())
  accountId String
  type      String   // DEPOSIT | WITHDRAWAL | FEE | PAYOUT
  amount    Decimal  @db.Decimal(18, 2)
  occurredAt DateTime
  note      String?
  account   TradingAccount @relation(fields: [accountId], references: [id], onDelete: Cascade)

  @@index([accountId, occurredAt])
}

model Journal {
  id        String          @id @default(cuid())
  userId    String
  name      String
  category  JournalCategory
  createdAt DateTime        @default(now())
  user      User            @relation(fields: [userId], references: [id], onDelete: Cascade)
  trades    Trade[]

  @@index([userId])
}

model Strategy {
  id        String   @id @default(cuid())
  userId    String
  name      String
  version   Int      @default(1)
  description String?
  createdAt DateTime @default(now())
  user      User     @relation(fields: [userId], references: [id], onDelete: Cascade)
  trades    Trade[]

  @@unique([userId, name, version])
}

model Trade {
  id             String      @id @default(cuid())
  userId         String
  accountId      String
  journalId      String?
  strategyId     String?

  symbol         String
  side           TradeSide
  status         TradeStatus @default(CLOSED)

  openedAt       DateTime
  closedAt       DateTime?

  volume         Decimal     @db.Decimal(18, 4)
  entryPrice     Decimal     @db.Decimal(18, 8)
  exitPrice      Decimal?    @db.Decimal(18, 8)
  stopLoss       Decimal?    @db.Decimal(18, 8)
  takeProfit     Decimal?    @db.Decimal(18, 8)

  grossPnl       Decimal?    @db.Decimal(18, 2)
  commission     Decimal     @db.Decimal(18, 2) @default(0)
  swap           Decimal     @db.Decimal(18, 2) @default(0)
  netPnl         Decimal?    @db.Decimal(18, 2)

  session        String?     // ASIA | LONDON | NEWYORK | SYDNEY
  timeframe      String?     // M15 | H1 | H4 | D1
  setupTag       String?
  mistakeTags    String[]    @default([])
  notes          String?

  // Provenance & dedup
  source         ImportSource @default(MANUAL)
  externalId     String?
  createdAt      DateTime    @default(now())
  updatedAt      DateTime    @updatedAt

  user           User        @relation(fields: [userId], references: [id], onDelete: Cascade)
  account        TradingAccount @relation(fields: [accountId], references: [id], onDelete: Cascade)
  journal        Journal?    @relation(fields: [journalId], references: [id], onDelete: SetNull)
  strategy       Strategy?   @relation(fields: [strategyId], references: [id], onDelete: SetNull)
  screenshots    Screenshot[]
  chartLinks     ChartLink[]
  aiAnalyses     AIAnalysis[]

  @@unique([accountId, externalId])
  @@index([userId, openedAt])
  @@index([accountId, symbol])
  @@index([userId, strategyId])
}

model Screenshot {
  id        String   @id @default(cuid())
  userId    String
  tradeId   String?
  key       String   // S3 object key
  mimeType  String
  size      Int
  caption   String?
  createdAt DateTime @default(now())
  user      User     @relation(fields: [userId], references: [id], onDelete: Cascade)
  trade     Trade?   @relation(fields: [tradeId], references: [id], onDelete: SetNull)
  @@index([userId])
  @@index([tradeId])
}

model ChartLink {
  id      String @id @default(cuid())
  tradeId String
  url     String
  caption String?
  trade   Trade  @relation(fields: [tradeId], references: [id], onDelete: Cascade)
  @@index([tradeId])
}

model AIAnalysis {
  id        String   @id @default(cuid())
  userId    String
  tradeId   String?
  kind      String   // TRADE_REVIEW | SCREENSHOT
  model     String
  prompt    String   @db.Text
  response  String   @db.Text
  createdAt DateTime @default(now())
  user      User     @relation(fields: [userId], references: [id], onDelete: Cascade)
  trade     Trade?   @relation(fields: [tradeId], references: [id], onDelete: Cascade)
  @@index([userId, createdAt])
}

model AuditEvent {
  id        String   @id @default(cuid())
  userId    String
  action    String
  metadata  Json?
  createdAt DateTime @default(now())
  user      User     @relation(fields: [userId], references: [id], onDelete: Cascade)
  @@index([userId, createdAt])
}

// NextAuth
model Account {
  id                String  @id @default(cuid())
  userId            String
  type              String
  provider          String
  providerAccountId String
  refresh_token     String? @db.Text
  access_token      String? @db.Text
  expires_at        Int?
  token_type        String?
  scope             String?
  id_token          String? @db.Text
  session_state     String?
  user User @relation(fields: [userId], references: [id], onDelete: Cascade)
  @@unique([provider, providerAccountId])
}

model Session {
  id           String   @id @default(cuid())
  sessionToken String   @unique
  userId       String
  expires      DateTime
  user User @relation(fields: [userId], references: [id], onDelete: Cascade)
}

model VerificationToken {
  identifier String
  token      String   @unique
  expires    DateTime
  @@unique([identifier, token])
}
import { PrismaClient } from "@prisma/client";

const globalForPrisma = globalThis as unknown as { prisma?: PrismaClient };
export const prisma =
  globalForPrisma.prisma ?? new PrismaClient({ log: ["warn", "error"] });
if (process.env.NODE_ENV !== "production") globalForPrisma.prisma = prisma;
import NextAuth from "next-auth";
import { PrismaAdapter } from "@auth/prisma-adapter";
import Credentials from "next-auth/providers/credentials";
import Google from "next-auth/providers/google";
import bcrypt from "bcryptjs";
import { prisma } from "./db";
import { z } from "zod";

const providers = [
  Credentials({
    name: "Email",
    credentials: {
      email: { label: "Email", type: "email" },
      password: { label: "Password", type: "password" },
    },
    async authorize(raw) {
      const parsed = z
        .object({ email: z.string().email(), password: z.string().min(8) })
        .safeParse(raw);
      if (!parsed.success) return null;
      const user = await prisma.user.findUnique({ where: { email: parsed.data.email } });
      if (!user?.passwordHash) return null;
      const ok = await bcrypt.compare(parsed.data.password, user.passwordHash);
      if (!ok) return null;
      return { id: user.id, email: user.email, name: user.name };
    },
  }),
];

if (process.env.GOOGLE_CLIENT_ID && process.env.GOOGLE_CLIENT_SECRET) {
  providers.push(
    Google({
      clientId: process.env.GOOGLE_CLIENT_ID,
      clientSecret: process.env.GOOGLE_CLIENT_SECRET,
    }) as any
  );
}

export const { handlers, auth, signIn, signOut } = NextAuth({
  adapter: PrismaAdapter(prisma) as any,
  session: { strategy: "jwt" },
  providers,
  pages: { signIn: "/login" },
  callbacks: {
    async jwt({ token, user }) {
      if (user) token.uid = user.id;
      return token;
    },
    async session({ session, token }) {
      if (session.user) (session.user as any).id = token.uid;
      return session;
    },
  },
});

export async function requireUser() {
  const session = await auth();
  if (!session?.user) throw new Response("Unauthorized", { status: 401 });
  return session.user as { id: string; email: string; name?: string | null };
}
export { auth as middleware } from "@/lib/auth";

export const config = {
  matcher: ["/dashboard/:path*", "/accounts/:path*", "/trades/:path*", "/import/:path*"],
};
import { z } from "zod";

export const AccountCreate = z.object({
  name: z.string().min(1).max(80),
  broker: z.string().max(80).optional(),
  currency: z.string().length(3).default("USD"),
  kind: z.enum(["LIVE", "DEMO", "PROP"]),
  initialBalance: z.string().regex(/^\d+(\.\d{1,2})?$/),
});

export const TradeCreate = z.object({
  accountId: z.string().cuid(),
  journalId: z.string().cuid().optional().nullable(),
  strategyId: z.string().cuid().optional().nullable(),
  symbol: z.string().min(1).max(20).toUpperCase(),
  side: z.enum(["BUY", "SELL"]),
  status: z.enum(["OPEN", "CLOSED"]).default("CLOSED"),
  openedAt: z.coerce.date(),
  closedAt: z.coerce.date().optional().nullable(),
  volume: z.string().regex(/^\d+(\.\d{1,4})?$/),
  entryPrice: z.string().regex(/^\d+(\.\d{1,8})?$/),
  exitPrice: z.string().regex(/^\d+(\.\d{1,8})?$/).optional().nullable(),
  stopLoss: z.string().regex(/^\d+(\.\d{1,8})?$/).optional().nullable(),
  takeProfit: z.string().regex(/^\d+(\.\d{1,8})?$/).optional().nullable(),
  grossPnl: z.string().regex(/^-?\d+(\.\d{1,2})?$/).optional().nullable(),
  commission: z.string().regex(/^-?\d+(\.\d{1,2})?$/).default("0"),
  swap: z.string().regex(/^-?\d+(\.\d{1,2})?$/).default("0"),
  session: z.string().max(20).optional().nullable(),
  timeframe: z.string().max(10).optional().nullable(),
  setupTag: z.string().max(40).optional().nullable(),
  notes: z.string().max(4000).optional().nullable(),
});

export type TradeCreateInput = z.infer<typeof TradeCreate>;
import { Prisma } from "@prisma/client";

export interface TradeRow {
  id: string;
  symbol: string;
  side: "BUY" | "SELL";
  status: "OPEN" | "CLOSED";
  openedAt: Date;
  closedAt: Date | null;
  volume: Prisma.Decimal;
  entryPrice: Prisma.Decimal;
  exitPrice: Prisma.Decimal | null;
  stopLoss: Prisma.Decimal | null;
  grossPnl: Prisma.Decimal | null;
  commission: Prisma.Decimal;
  swap: Prisma.Decimal;
  netPnl: Prisma.Decimal | null;
  session: string | null;
  timeframe: string | null;
  setupTag: string | null;
  strategyId: string | null;
  accountId: string;
}

export interface Metrics {
  trades: number;
  wins: number;
  losses: number;
  breakeven: number;
  winRate: number;              // 0..1
  grossProfit: number;
  grossLoss: number;            // negative
  netProfit: number;
  avgWin: number;
  avgLoss: number;              // negative
  profitFactor: number | null;  // null when no losses
  expectancy: number;           // avg net P&L per closed trade
  avgR: number | null;          // requires stopLoss + netPnl
  maxConsecutiveWins: number;
  maxConsecutiveLosses: number;
  maxDrawdown: number;          // negative number, computed on netPnl equity curve
  equityCurve: { ts: string; equity: number; dd: number }[];
}

const num = (d: Prisma.Decimal | null | undefined) => (d ? Number(d) : 0);

/**
 * Compute metrics from *closed* trades only. Open trades are ignored.
 * Max drawdown is computed from the realized P&L equity curve starting at 0 —
 * this is NOT account drawdown (account drawdown requires balance history
 * including deposits/withdrawals, which is not used here by design).
 */
export function computeMetrics(trades: TradeRow[], startingEquity = 0): Metrics {
  const closed = trades
    .filter((t) => t.status === "CLOSED" && t.netPnl !== null)
    .sort((a, b) => (a.closedAt ?? a.openedAt).getTime() - (b.closedAt ?? b.openedAt).getTime());

  const net = closed.map((t) => num(t.netPnl));
  const winsArr = net.filter((n) => n > 0);
  const lossArr = net.filter((n) => n < 0);
  const beCount = net.filter((n) => n === 0).length;

  const grossProfit = winsArr.reduce((a, b) => a + b, 0);
  const grossLoss = lossArr.reduce((a, b) => a + b, 0);
  const netProfit = grossProfit + grossLoss;

  const avgWin = winsArr.length ? grossProfit / winsArr.length : 0;
  const avgLoss = lossArr.length ? grossLoss / lossArr.length : 0;

  const winRate = closed.length ? winsArr.length / closed.length : 0;

  let pf: number | null = null;
  if (lossArr.length && grossLoss !== 0) pf = grossProfit / Math.abs(grossLoss);

  const expectancy = closed.length ? netProfit / closed.length : 0;

  // R multiples — only for trades that have a stop and an entry
  const rMultiples: number[] = [];
  for (const t of closed) {
    if (!t.stopLoss || !t.exitPrice) continue;
    const entry = num(t.entryPrice);
    const stop = num(t.stopLoss);
    const exit = num(t.exitPrice);
    const risk = Math.abs(entry - stop);
    if (risk === 0) continue;
    const r = ((t.side === "BUY" ? exit - entry : entry - exit) / risk);
    rMultiples.push(r);
  }
  const avgR = rMultiples.length ? rMultiples.reduce((a, b) => a + b, 0) / rMultiples.length : null;

  // Consecutive streaks
  let cw = 0, cl = 0, mw = 0, ml = 0;
  for (const n of net) {
    if (n > 0) { cw++; cl = 0; mw = Math.max(mw, cw); }
    else if (n < 0) { cl++; cw = 0; ml = Math.max(ml, cl); }
    else { cw = 0; cl = 0; }
  }

  // Equity curve + drawdown
  let equity = startingEquity;
  let peak = startingEquity;
  let maxDD = 0;
  const equityCurve: { ts: string; equity: number; dd: number }[] = [];
  for (let i = 0; i < closed.length; i++) {
    equity += net[i];
    if (equity > peak) peak = equity;
    const dd = peak === 0 ? 0 : (equity - peak) / Math.abs(peak || 1);
    if (equity - peak < maxDD) maxDD = equity - peak;
    equityCurve.push({
      ts: (closed[i].closedAt ?? closed[i].openedAt).toISOString(),
      equity,
      dd,
    });
  }

  return {
    trades: closed.length,
    wins: winsArr.length,
    losses: lossArr.length,
    breakeven: beCount,
    winRate,
    grossProfit,
    grossLoss,
    netProfit,
    avgWin,
    avgLoss,
    profitFactor: pf,
    expectancy,
    avgR,
    maxConsecutiveWins: mw,
    maxConsecutiveLosses: ml,
    maxDrawdown: maxDD,
    equityCurve,
  };
}

export interface GroupBreakdown {
  key: string;
  metrics: Metrics;
}

export function groupBy<T extends keyof TradeRow>(
  trades: TradeRow[],
  field: T
): GroupBreakdown[] {
  const buckets = new Map<string, TradeRow[]>();
  for (const t of trades) {
    const key = String(t[field] ?? "—");
    if (!buckets.has(key)) buckets.set(key, []);
    buckets.get(key)!.push(t);
  }
  return [...buckets.entries()]
    .map(([key, rows]) => ({ key, metrics: computeMetrics(rows) }))
    .sort((a, b) => b.metrics.netProfit - a.metrics.netProfit);
}

/**
 * Net P&L per trade is derived, not stored blindly:
 *   netPnl = grossPnl + commission + swap
 * Commission is typically negative in imports; we accept whatever sign the
 * broker supplies and let the user's import mapping decide.
 */
export function deriveNetPnl(
  grossPnl: Prisma.Decimal | null,
  commission: Prisma.Decimal,
  swap: Prisma.Decimal
): Prisma.Decimal | null {
  if (grossPnl === null) return null;
  return new Prisma.Decimal(grossPnl).plus(commission).plus(swap);
}
import { describe, it, expect } from "vitest";
import { Prisma } from "@prisma/client";
import { computeMetrics, type TradeRow } from "./analytics";

const D = (n: number) => new Prisma.Decimal(n);
const mk = (net: number, extra: Partial<TradeRow> = {}): TradeRow => ({
  id: "x", symbol: "XAUUSD", side: "BUY", status: "CLOSED",
  openedAt: new Date(), closedAt: new Date(),
  volume: D(1), entryPrice: D(100), exitPrice: D(100),
  stopLoss: null, grossPnl: D(net), commission: D(0), swap: D(0),
  netPnl: D(net), session: null, timeframe: null, setupTag: null,
  strategyId: null, accountId: "a", ...extra,
});

describe("computeMetrics", () => {
  it("empty", () => {
    const m = computeMetrics([]);
    expect(m.trades).toBe(0);
    expect(m.winRate).toBe(0);
    expect(m.profitFactor).toBeNull();
  });
  it("basic win/loss", () => {
    const m = computeMetrics([mk(100), mk(-50), mk(200), mk(-100)]);
    expect(m.trades).toBe(4);
    expect(m.wins).toBe(2);
    expect(m.losses).toBe(2);
    expect(m.winRate).toBe(0.5);
    expect(m.grossProfit).toBe(300);
    expect(m.grossLoss).toBe(-150);
    expect(m.netProfit).toBe(150);
    expect(m.profitFactor).toBe(2);
    expect(m.expectancy).toBe(37.5);
  });
  it("ignores open trades", () => {
    const m = computeMetrics([mk(100), { ...mk(0), status: "OPEN", netPnl: null }]);
    expect(m.trades).toBe(1);
  });
  it("R multiple", () => {
    const t = mk(0, { stopLoss: D(90), exitPrice: D(120), side: "BUY" });
    const m = computeMetrics([t]);
    expect(m.avgR).toBeCloseTo(2);
  });
});
import { NextResponse } from "next/server";
import { ZodError } from "zod";

export function ok(data: unknown, init?: ResponseInit) {
  return NextResponse.json(data, init);
}
export function bad(msg: string, status = 400) {
  return NextResponse.json({ error: msg }, { status });
}
export function handle(err: unknown) {
  if (err instanceof ZodError) return bad(err.issues.map(i => i.message).join("; "), 422);
  if (err instanceof Response) return err;
  console.error(err);
  return bad("Internal error", 500);
}
import { handlers } from "@/lib/auth";
export const { GET, POST } = handlers;
import bcrypt from "bcryptjs";
import { z } from "zod";
import { prisma } from "@/lib/db";
import { ok, handle, bad } from "@/lib/api";

const Body = z.object({
  email: z.string().email(),
  password: z.string().min(8).max(128),
  name: z.string().min(1).max(80).optional(),
});

export async function POST(req: Request) {
  try {
    const body = Body.parse(await req.json());
    const existing = await prisma.user.findUnique({ where: { email: body.email } });
    if (existing) return bad("Email already registered", 409);
    const hash = await bcrypt.hash(body.password, 12);
    const user = await prisma.user.create({
      data: { email: body.email, name: body.name, passwordHash: hash },
      select: { id: true, email: true },
    });
    return ok(user, { status: 201 });
  } catch (e) { return handle(e); }
}
import { prisma } from "@/lib/db";
import { requireUser } from "@/lib/auth";
import { AccountCreate } from "@/lib/validation";
import { Prisma } from "@prisma/client";
import { ok, handle } from "@/lib/api";

export async function GET() {
  try {
    const u = await requireUser();
    const accounts = await prisma.tradingAccount.findMany({
      where: { userId: u.id, isArchived: false },
      orderBy: { createdAt: "asc" },
    });
    return ok(accounts);
  } catch (e) { return handle(e); }
}

export async function POST(req: Request) {
  try {
    const u = await requireUser();
    const input = AccountCreate.parse(await req.json());
    const account = await prisma.tradingAccount.create({
      data: {
        userId: u.id,
        name: input.name,
        broker: input.broker,
        currency: input.currency,
        kind: input.kind,
        initialBalance: new Prisma.Decimal(input.initialBalance),
      },
    });
    await prisma.auditEvent.create({
      data: { userId: u.id, action: "account.create", metadata: { accountId: account.id } },
    });
    return ok(account, { status: 201 });
  } catch (e) { return handle(e); }
}
import { prisma } from "@/lib/db";
import { requireUser } from "@/lib/auth";
import { TradeCreate } from "@/lib/validation";
import { deriveNetPnl } from "@/lib/analytics";
import { Prisma } from "@prisma/client";
import { ok, handle, bad } from "@/lib/api";

export async function GET(req: Request) {
  try {
    const u = await requireUser();
    const url = new URL(req.url);
    const accountId = url.searchParams.get("accountId") ?? undefined;
    const limit = Math.min(Number(url.searchParams.get("limit") ?? 100), 500);
    const trades = await prisma.trade.findMany({
      where: { userId: u.id, ...(accountId ? { accountId } : {}) },
      orderBy: { openedAt: "desc" },
      take: limit,
      include: { screenshots: true, chartLinks: true },
    });
    return ok(trades);
  } catch (e) { return handle(e); }
}

export async function POST(req: Request) {
  try {
    const u = await requireUser();
    const input = TradeCreate.parse(await req.json());

    // Verify account belongs to user (IDOR protection)
    const account = await prisma.tradingAccount.findFirst({
      where: { id: input.accountId, userId: u.id },
    });
    if (!account) return bad("Account not found", 404);

    const gross = input.grossPnl != null ? new Prisma.Decimal(input.grossPnl) : null;
    const comm = new Prisma.Decimal(input.commission);
    const swap = new Prisma.Decimal(input.swap);
    const net = deriveNetPnl(gross, comm, swap);

    const trade = await prisma.trade.create({
      data: {
        userId: u.id,
        accountId: input.accountId,
        journalId: input.journalId ?? null,
        strategyId: input.strategyId ?? null,
        symbol: input.symbol,
        side: input.side,
        status: input.status,
        openedAt: input.openedAt,
        closedAt: input.closedAt ?? null,
        volume: new Prisma.Decimal(input.volume),
        entryPrice: new Prisma.Decimal(input.entryPrice),
        exitPrice: input.exitPrice != null ? new Prisma.Decimal(input.exitPrice) : null,
        stopLoss: input.stopLoss != null ? new Prisma.Decimal(input.stopLoss) : null,
        takeProfit: input.takeProfit != null ? new Prisma.Decimal(input.takeProfit) : null,
        grossPnl: gross,
        commission: comm,
        swap,
        netPnl: net,
        session: input.session ?? null,
        timeframe: input.timeframe ?? null,
        setupTag: input.setupTag ?? null,
        notes: input.notes ?? null,
        source: "MANUAL",
      },
    });
    return ok(trade, { status: 201 });
  } catch (e) { return handle(e); }
}
import { prisma } from "@/lib/db";
import { requireUser } from "@/lib/auth";
import { TradeCreate } from "@/lib/validation";
import { deriveNetPnl } from "@/lib/analytics";
import { Prisma } from "@prisma/client";
import { ok, handle, bad } from "@/lib/api";

async function ownedTrade(id: string, userId: string) {
  return prisma.trade.findFirst({ where: { id, userId } });
}

export async function GET(_: Request, { params }: { params: { id: string } }) {
  try {
    const u = await requireUser();
    const t = await prisma.trade.findFirst({
      where: { id: params.id, userId: u.id },
      include: { screenshots: true, chartLinks: true, aiAnalyses: true },
    });
    if (!t) return bad("Not found", 404);
    return ok(t);
  } catch (e) { return handle(e); }
}

export async function PATCH(req: Request, { params }: { params: { id: string } }) {
  try {
    const u = await requireUser();
    if (!(await ownedTrade(params.id, u.id))) return bad("Not found", 404);
    const input = TradeCreate.partial().parse(await req.json());

    const gross = input.grossPnl != null ? new Prisma.Decimal(input.grossPnl) : undefined;
    const comm = input.commission != null ? new Prisma.Decimal(input.commission) : undefined;
    const swap = input.swap != null ? new Prisma.Decimal(input.swap) : undefined;

    const current = await prisma.trade.findUniqueOrThrow({ where: { id: params.id } });
    const net = deriveNetPnl(
      gross ?? current.grossPnl,
      comm ?? current.commission,
      swap ?? current.swap
    );

    const t = await prisma.trade.update({
      where: { id: params.id },
      data: {
        ...(input.symbol && { symbol: input.symbol }),
        ...(input.side && { side: input.side }),
        ...(input.status && { status: input.status }),
        ...(input.openedAt && { openedAt: input.openedAt }),
        ...(input.closedAt !== undefined && { closedAt: input.closedAt }),
        ...(input.volume && { volume: new Prisma.Decimal(input.volume) }),
        ...(input.entryPrice && { entryPrice: new Prisma.Decimal(input.entryPrice) }),
        ...(input.exitPrice !== undefined && {
          exitPrice: input.exitPrice ? new Prisma.Decimal(input.exitPrice) : null,
        }),
        ...(input.stopLoss !== undefined && {
          stopLoss: input.stopLoss ? new Prisma.Decimal(input.stopLoss) : null,
        }),
        ...(input.takeProfit !== undefined && {
          takeProfit: input.takeProfit ? new Prisma.Decimal(input.takeProfit) : null,
        }),
        ...(gross !== undefined && { grossPnl: gross }),
        ...(comm !== undefined && { commission: comm }),
        ...(swap !== undefined && { swap }),
        netPnl: net,
        ...(input.session !== undefined && { session: input.session }),
        ...(input.timeframe !== undefined && { timeframe: input.timeframe }),
        ...(input.setupTag !== undefined && { setupTag: input.setupTag }),
        ...(input.notes !== undefined && { notes: input.notes }),
      },
    });
    return ok(t);
  } catch (e) { return handle(e); }
}

export async function DELETE(_: Request, { params }: { params: { id: string } }) {
  try {
    const u = await requireUser();
    if (!(await ownedTrade(params.id, u.id))) return bad("Not found", 404);
    await prisma.trade.delete({ where: { id: params.id } });
    return ok({ deleted: true });
  } catch (e) { return handle(e); }
}
import { prisma } from "@/lib/db";
import { requireUser } from "@/lib/auth";
import { computeMetrics, groupBy, type TradeRow } from "@/lib/analytics";
import { ok, handle } from "@/lib/api";

export async function GET(req: Request) {
  try {
    const u = await requireUser();
    const url = new URL(req.url);
    const accountId = url.searchParams.get("accountId") ?? undefined;

    const trades = await prisma.trade.findMany({
      where: { userId: u.id, ...(accountId ? { accountId } : {}) },
    });

    const rows: TradeRow[] = trades.map((t) => ({
      id: t.id, symbol: t.symbol, side: t.side, status: t.status,
      openedAt: t.openedAt, closedAt: t.closedAt,
      volume: t.volume, entryPrice: t.entryPrice, exitPrice: t.exitPrice,
      stopLoss: t.stopLoss, grossPnl: t.grossPnl, commission: t.commission,
      swap: t.swap, netPnl: t.netPnl, session: t.session, timeframe: t.timeframe,
      setupTag: t.setupTag, strategyId: t.strategyId, accountId: t.accountId,
    }));

    const overall = computeMetrics(rows);
    const bySymbol = groupBy(rows, "symbol");
    const bySession = groupBy(rows, "session");
    const bySetup = groupBy(rows, "setupTag");
    const bySide = groupBy(rows, "side");

    return ok({ overall, bySymbol, bySession, bySetup, bySide });
  } catch (e) { return handle(e); }
}
import { prisma } from "@/lib/db";
import { requireUser } from "@/lib/auth";
import { Prisma } from "@prisma/client";
import { ok, handle, bad } from "@/lib/api";

/** Minimal CSV parser that respects quoted fields. No external dep. */
function parseCsv(text: string): string[][] {
  const rows: string[][] = [];
  let row: string[] = [];
  let cur = "";
  let inQuotes = false;
  for (let i = 0; i < text.length; i++) {
    const ch = text[i];
    if (inQuotes) {
      if (ch === '"' && text[i + 1] === '"') { cur += '"'; i++; }
      else if (ch === '"') inQuotes = false;
      else cur += ch;
    } else {
      if (ch === '"') inQuotes = true;
      else if (ch === ",") { row.push(cur); cur = ""; }
      else if (ch === "\n") { row.push(cur); rows.push(row); row = []; cur = ""; }
      else if (ch === "\r") { /* skip */ }
      else cur += ch;
    }
  }
  if (cur || row.length) { row.push(cur); rows.push(row); }
  return rows.filter((r) => r.some((c) => c.trim() !== ""));
}

const FIELD_ALIASES: Record<string, string[]> = {
  symbol: ["symbol", "instrument", "pair"],
  side: ["type", "side", "action"],
  volume: ["volume", "lots", "size"],
  entryPrice: ["open price", "entry", "openprice", "price open"],
  exitPrice: ["close price", "exit", "closeprice", "price close"],
  openedAt: ["open time", "opentime", "time open", "entry time"],
  closedAt: ["close time", "closetime", "time close", "exit time"],
  grossPnl: ["profit", "pnl", "gross p/l", "gross profit"],
  commission: ["commission", "comm"],
  swap: ["swap", "rollover"],
  externalId: ["ticket", "deal id", "order", "id"],
  stopLoss: ["s/l", "sl", "stop loss"],
  takeProfit: ["t/p", "tp", "take profit"],
};

function findIndex(header: string[], names: string[]): number {
  const lower = header.map((h) => h.trim().toLowerCase());
  for (const n of names) {
    const idx = lower.indexOf(n);
    if (idx >= 0) return idx;
  }
  return -1;
}

function parseNumber(s: string | undefined): string | null {
  if (!s) return null;
  const cleaned = s.replace(/[^\d.\-]/g, "");
  if (!cleaned || cleaned === "-" || cleaned === ".") return null;
  return cleaned;
}

function parseDate(s: string | undefined): Date | null {
  if (!s) return null;
  const d = new Date(s.trim());
  return isNaN(d.getTime()) ? null : d;
}

export async function POST(req: Request) {
  try {
    const u = await requireUser();
    const form = await req.formData();
    const file = form.get("file");
    const accountId = String(form.get("accountId") ?? "");
    if (!(file instanceof File)) return bad("file required", 422);
    if (!accountId) return bad("accountId required", 422);
    if (file.size > 10 * 1024 * 1024) return bad("file too large", 413);

    const account = await prisma.tradingAccount.findFirst({
      where: { id: accountId, userId: u.id },
    });
    if (!account) return bad("Account not found", 404);

    const text = await file.text();
    const rows = parseCsv(text);
    if (rows.length < 2) return bad("CSV has no data rows", 422);

    const header = rows[0];
    const idx: Record<string, number> = {};
    for (const [field, aliases] of Object.entries(FIELD_ALIASES)) {
      idx[field] = findIndex(header, aliases);
    }

    const missing: string[] = [];
    for (const req of ["symbol", "side", "volume", "entryPrice", "openedAt", "grossPnl"]) {
      if (idx[req] < 0) missing.push(req);
    }
    if (missing.length) return bad(`Could not map columns: ${missing.join(", ")}. Header was: ${header.join(" | ")}`, 422);

    let imported = 0, skipped = 0;
    const errors: string[] = [];

    for (let i = 1; i < rows.length; i++) {
      const r = rows[i];
      const get = (f: string) => (idx[f] >= 0 ? r[idx[f]] : undefined);

      const symbolRaw = get("symbol")?.trim().toUpperCase();
      const sideRaw = get("side")?.trim().toLowerCase();
      const volume = parseNumber(get("volume"));
      const entry = parseNumber(get("entryPrice"));
      const openedAt = parseDate(get("openedAt"));
      const gross = parseNumber(get("grossPnl"));

      if (!symbolRaw || !sideRaw || !volume || !entry || !openedAt || gross === null) {
        skipped++; errors.push(`row ${i + 1}: missing required field`); continue;
      }
      const side = sideRaw.startsWith("b") ? "BUY" : sideRaw.startsWith("s") ? "SELL" : null;
      if (!side) { skipped++; errors.push(`row ${i + 1}: unknown side "${sideRaw}"`); continue; }

      const externalId = get("externalId")?.trim() || null;
      try {
        await prisma.trade.create({
          data: {
            userId: u.id,
            accountId: account.id,
            symbol: symbolRaw,
            side,
            status: "CLOSED",
            openedAt,
            closedAt: parseDate(get("closedAt")) ?? openedAt,
            volume: new Prisma.Decimal(volume),
            entryPrice: new Prisma.Decimal(entry),
            exitPrice: parseNumber(get("exitPrice")) ? new Prisma.Decimal(parseNumber(get("exitPrice"))!) : null,
            stopLoss: parseNumber(get("stopLoss")) ? new Prisma.Decimal(parseNumber(get("stopLoss"))!) : null,
            takeProfit: parseNumber(get("takeProfit")) ? new Prisma.Decimal(parseNumber(get("takeProfit"))!) : null,
            grossPnl: new Prisma.Decimal(gross),
            commission: new Prisma.Decimal(parseNumber(get("commission")) ?? 0),
            swap: new Prisma.Decimal(parseNumber(get("swap")) ?? 0),
            netPnl: new Prisma.Decimal(gross)
              .plus(new Prisma.Decimal(parseNumber(get("commission")) ?? 0))
              .plus(new Prisma.Decimal(parseNumber(get("swap")) ?? 0)),
            source: "CSV_STATEMENT",
            externalId,
          },
        });
        imported++;
      } catch (e: any) {
        if (e?.code === "P2002") { skipped++; errors.push(`row ${i + 1}: duplicate`); }
        else { skipped++; errors.push(`row ${i + 1}: ${String(e?.message ?? e)}`); }
      }
    }

    return ok({ imported, skipped, errors: errors.slice(0, 50) });
  } catch (e) { return handle(e); }
}
// Real AI integration. If the provider is not configured, callers must
// surface the unavailable state — never fabricate analysis.
import Anthropic from "@anthropic-ai/sdk";

export function isAiConfigured(): boolean {
  if (process.env.AI_PROVIDER === "anthropic") return !!process.env.ANTHROPIC_API_KEY;
  if (process.env.AI_PROVIDER === "openai") return !!process.env.OPENAI_API_KEY;
  return false;
}

const SYSTEM = `You are NuelTrades AI, a disciplined trading coach.
Rules:
- Only analyze what is present in the provided trade data. Never invent prices,
  candle data, or market context that is not provided.
- Separate OBSERVATIONS (facts from the data) from HYPOTHESES (interpretations).
- State uncertainty explicitly. If data is insufficient, say so.
- Do not recommend specific position sizes or guarantee outcomes.
- Reply in structured Markdown with sections: Summary, Execution, Risk,
  Mistakes, Lessons, Suggested Tags, Follow-up Questions.`;

export async function analyzeTrade(payload: {
  trade: Record<string, unknown>;
  recentTrades?: Record<string, unknown>[];
}): Promise<{ text: string; model: string }> {
  if (process.env.AI_PROVIDER !== "anthropic" || !process.env.ANTHROPIC_API_KEY) {
    throw new Error("AI_NOT_CONFIGURED");
  }
  const client = new Anthropic({ apiKey: process.env.ANTHROPIC_API_KEY });
  const model = process.env.AI_MODEL || "claude-3-5-sonnet-20241022";

  const userPrompt = [
    "Analyze this single trade.",
    "TRADE:", JSON.stringify(payload.trade, null, 2),
    payload.recentTrades?.length
      ? `CONTEXT — recent trades from same account:\n${JSON.stringify(payload.recentTrades, null, 2)}`
      : "",
    "Remember: distinguish observations from hypotheses and mark uncertainty.",
  ].filter(Boolean).join("\n\n");

  const res = await client.messages.create({
    model,
    max_tokens: 1200,
    system: SYSTEM,
    messages: [{ role: "user", content: userPrompt }],
  });
  const text = res.content
    .map((c) => (c.type === "text" ? c.text : ""))
    .join("\n")
    .trim();
  return { text, model };
}
import { prisma } from "@/lib/db";
import { requireUser } from "@/lib/auth";
import { analyzeTrade, isAiConfigured } from "@/lib/ai";
import { ok, handle, bad } from "@/lib/api";
import { z } from "zod";

const Body = z.object({ tradeId: z.string().cuid() });

export async function POST(req: Request) {
  try {
    const u = await requireUser();
    if (!isAiConfigured()) return bad("AI provider not configured on this server", 503);

    const { tradeId } = Body.parse(await req.json());
    const trade = await prisma.trade.findFirst({ where: { id: tradeId, userId: u.id } });
    if (!trade) return bad("Trade not found", 404);

    const recent = await prisma.trade.findMany({
      where: { userId: u.id, accountId: trade.accountId, id: { not: trade.id } },
      orderBy: { openedAt: "desc" },
      take: 10,
    });

    const { text, model } = await analyzeTrade({
      trade: JSON.parse(JSON.stringify(trade)),
      recentTrades: JSON.parse(JSON.stringify(recent)),
    });

    const analysis = await prisma.aIAnalysis.create({
      data: {
        userId: u.id,
        tradeId: trade.id,
        kind: "TRADE_REVIEW",
        model,
        prompt: "See route logs",
        response: text,
      },
    });
    return ok(analysis);
  } catch (e) { return handle(e); }
}
// layout.tsx
import "./globals.css";
import type { Metadata } from "next";
export const metadata: Metadata = { title: "NuelTrades AI" };
export default function RootLayout({ children }: { children: React.ReactNode }) {
  return (<html lang="en"><body>{children}</body></html>);
}
// page.tsx
import Link from "next/link";
export default function Home() {
  return (
    <main className="min-h-screen flex flex-col items-center justify-center px-6">
      <h1 className="text-4xl font-semibold mb-3">NuelTrades AI</h1>
      <p className="text-muted mb-8 text-center max-w-md">
        AI-powered trading journal &amp; analytics.
      </p>
      <div className="flex gap-3">
        <Link href="/login" className="px-5 py-2 border border-border rounded-md">Sign in</Link>
        <Link href="/register" className="px-5 py-2 bg-pos text-black rounded-md font-medium">Create account</Link>
      </div>
    </main>
  );
}
import Link from "next/link";
const items = [
  ["Dashboard", "/dashboard"],
  ["Accounts", "/accounts"],
  ["Trades", "/trades"],
  ["Import", "/import"],
];
export function Nav() {
  return (
    <nav className="border-b border-border bg-panel">
      <div className="max-w-6xl mx-auto px-4 h-14 flex items-center gap-6">
        <Link href="/dashboard" className="font-semibold">NuelTrades AI</Link>
        <ul className="flex gap-4 text-sm text-muted">
          {items.map(([label, href]) => (
            <li key={href}><Link href={href} className="hover:text-white">{label}</Link></li>
          ))}
        </ul>
      </div>
    </nav>
  );
}
import { requireUser } from "@/lib/auth";
import { prisma } from "@/lib/db";
import { computeMetrics, type TradeRow } from "@/lib/analytics";
import { Nav } from "@/components/Nav";
import { MetricCard } from "@/components/MetricCard";
import { EquityCurve } from "@/components/EquityCurve";

export default async function Dashboard() {
  const u = await requireUser();
  const trades = await prisma.trade.findMany({ where: { userId: u.id } });
  const rows: TradeRow[] = trades.map((t) => ({ ...t } as any));
  const m = computeMetrics(rows);

  return (
    <>
      <Nav />
      <main className="max-w-6xl mx-auto p-6">
        <h1 className="text-2xl font-semibold mb-6">Dashboard</h1>
        {m.trades === 0 ? (
          <div className="border border-border rounded-lg p-8 text-center text-muted">
            No closed trades yet. Add a trade or import a CSV statement to begin.
          </div>
        ) : (
          <>
            <div className="grid grid-cols-2 md:grid-cols-4 gap-3 mb-6">
              <MetricCard label="Net P&L" value={m.netProfit.toFixed(2)} tone={m.netProfit >= 0 ? "pos" : "neg"} />
              <MetricCard label="Win rate" value={(m.winRate * 100).toFixed(1) + "%"} />
              <MetricCard label="Profit factor" value={m.profitFactor?.toFixed(2) ?? "—"} />
              <MetricCard label="Expectancy" value={m.expectancy.toFixed(2)} tone={m.expectancy >= 0 ? "pos" : "neg"} />
              <MetricCard label="Trades" value={String(m.trades)} />
              <MetricCard label="Avg win" value={m.avgWin.toFixed(2)} tone="pos" />
              <MetricCard label="Avg loss" value={m.avgLoss.toFixed(2)} tone="neg" />
              <MetricCard label="Max consec. losses" value={String(m.maxConsecutiveLosses)} />
            </div>
            <div className="border border-border rounded-lg p-4">
              <h2 className="text-sm text-muted mb-2">Realized equity curve (starting from 0)</h2>
              <EquityCurve data={m.equityCurve} />
            </div>
          </>
        )}
      </main>
    </>
  );
}
// MetricCard.tsx
export function MetricCard({ label, value, tone }: { label: string; value: string; tone?: "pos" | "neg" }) {
  const color = tone === "pos" ? "text-pos" : tone === "neg" ? "text-neg" : "";
  return (
    <div className="border border-border rounded-lg p-4">
      <div className="text-xs text-muted mb-1">{label}</div>
      <div className={`text-xl font-semibold ${color}`}>{value}</div>
    </div>
  );
}
// EquityCurve.tsx
"use client";
import { LineChart, Line, XAxis, YAxis, Tooltip, ResponsiveContainer, ReferenceLine } from "recharts";
export function EquityCurve({ data }: { data: { ts: string; equity: number; dd: number }[] }) {
  return (
    <div className="h-64">
      <ResponsiveContainer>
        <LineChart data={data}>
          <XAxis dataKey="ts" tickFormatter={(v) => new Date(v).toLocaleDateString()} stroke="#8b8b93" fontSize={10} />
          <YAxis stroke="#8b8b93" fontSize={10} />
          <Tooltip contentStyle={{ background: "#131316", border: "1px solid #26262b" }} />
          <ReferenceLine y={0} stroke="#26262b" />
          <Line type="monotone" dataKey="equity" stroke="#22c55e" dot={false} strokeWidth={2} />
        </LineChart>
      </ResponsiveContainer>
    </div>
  );
}
// login/page.tsx
"use client";
import { useState } from "react";
import { signIn } from "next-auth/react";
import { useRouter } from "next/navigation";

export default function Login() {
  const r = useRouter();
  const [email, setEmail] = useState(""); const [password, setPassword] = useState("");
  const [err, setErr] = useState<string | null>(null); const [busy, setBusy] = useState(false);

  async function submit(e: React.FormEvent) {
    e.preventDefault(); setBusy(true); setErr(null);
    const res = await signIn("credentials", { email, password, redirect: false });
    setBusy(false);
    if (!res || res.error) { setErr("Invalid email or password"); return; }
    r.push("/dashboard");
  }
  return (
    <main className="min-h-screen grid place-items-center">
      <form onSubmit={submit} className="w-80 flex flex-col gap-3">
        <h1 className="text-xl font-semibold mb-2">Sign in</h1>
        <input placeholder="Email" type="email" value={email} onChange={(e) => setEmail(e.target.value)} required />
        <input placeholder="Password" type="password" value={password} onChange={(e) => setPassword(e.target.value)} required minLength={8} />
        {err && <div className="text-neg text-sm">{err}</div>}
        <button disabled={busy} className="bg-pos text-black rounded-md py-2 font-medium">
          {busy ? "Signing in…" : "Sign in"}
        </button>
      </form>
    </main>
  );
}
// register/page.tsx
"use client";
import { useState } from "react";
import { useRouter } from "next/navigation";

export default function Register() {
  const r = useRouter();
  const [email, setEmail] = useState(""); const [password, setPassword] = useState("");
  const [err, setErr] = useState<string | null>(null); const [busy, setBusy] = useState(false);
  async function submit(e: React.FormEvent) {
    e.preventDefault(); setBusy(true); setErr(null);
    const res = await fetch("/api/auth/register", {
      method: "POST", headers: { "content-type": "application/json" },
      body: JSON.stringify({ email, password }),
    });
    setBusy(false);
    if (!res.ok) { const j = await res.json().catch(() => ({})); setErr(j.error ?? "Registration failed"); return; }
    r.push("/login");
  }
  return (
    <main className="min-h-screen grid place-items-center">
      <form onSubmit={submit} className="w-80 flex flex-col gap-3">
        <h1 className="text-xl font-semibold mb-2">Create account</h1>
        <input placeholder="Email" type="email" value={email} onChange={(e) => setEmail(e.target.value)} required />
        <input placeholder="Password (min 8)" type="password" value={password} onChange={(e) => setPassword(e.target.value)} required minLength={8} />
        {err && <div className="text-neg text-sm">{err}</div>}
        <button disabled={busy} className="bg-pos text-black rounded-md py-2 font-medium">
          {busy ? "Creating…" : "Create account"}
        </button>
      </form>
    </main>
  );
}
"use client";
import { useEffect, useState } from "react";
import { Nav } from "@/components/Nav";

export default function Accounts() {
  const [accounts, setAccounts] = useState<any[]>([]);
  const [form, setForm] = useState({ name: "", broker: "", kind: "LIVE", currency: "USD", initialBalance: "0" });
  const [err, setErr] = useState<string | null>(null);

  async function load() {
    const res = await fetch("/api/accounts"); setAccounts(await res.json());
  }
  useEffect(() => { load(); }, []);

  async function create(e: React.FormEvent) {
    e.preventDefault(); setErr(null);
    const res = await fetch("/api/accounts", {
      method: "POST", headers: { "content-type": "application/json" }, body: JSON.stringify(form),
    });
    if (!res.ok) { const j = await res.json().catch(() => ({})); setErr(j.error ?? "Failed"); return; }
    setForm({ name: "", broker: "", kind: "LIVE", currency: "USD", initialBalance: "0" });
    load();
  }

  return (<><Nav /><main className="max-w-4xl mx-auto p-6">
    <h1 className="text-2xl font-semibold mb-4">Accounts</h1>
    <form onSubmit={create} className="grid grid-cols-2 gap-2 mb-6 max-w-lg">
      <input placeholder="Name" value={form.name} onChange={(e) => setForm({ ...form, name: e.target.value })} required />
      <input placeholder="Broker" value={form.broker} onChange={(e) => setForm({ ...form, broker: e.target.value })} />
      <select value={form.kind} onChange={(e) => setForm({ ...form, kind: e.target.value })}>
        <option>LIVE</option><option>DEMO</option><option>PROP</option>
      </select>
      <input placeholder="Currency" value={form.currency} onChange={(e) => setForm({ ...form, currency: e.target.value.toUpperCase() })} maxLength={3} />
      <input placeholder="Initial balance" value={form.initialBalance} onChange={(e) => setForm({ ...form, initialBalance: e.target.value })} pattern="^\d+(\.\d{1,2})?$" />
      <button className="bg-pos text-black rounded-md py-2 font-medium">Create account</button>
      {err && <div className="col-span-2 text-neg text-sm">{err}</div>}
    </form>
    <div className="border border-border rounded-lg divide-y divide-border">
      {accounts.length === 0 && <div className="p-4 text-muted text-sm">No accounts yet.</div>}
      {accounts.map((a) => (
        <div key={a.id} className="p-3 flex justify-between">
          <div><div className="font-medium">{a.name}</div><div className="text-xs text-muted">{a.kind} · {a.broker ?? "—"} · {a.currency}</div></div>
          <div className="text-sm text-muted">Start {Number(a.initialBalance).toFixed(2)}</div>
        </div>
      ))}
    </div>
  </main></>);
}
"use client";
import { useEffect, useState } from "react";
import { useRouter } from "next/navigation";
import { Nav } from "@/components/Nav";

export default function NewTrade() {
  const r = useRouter();
  const [accounts, setAccounts] = useState<any[]>([]);
  const [err, setErr] = useState<string | null>(null);
  const [busy, setBusy] = useState(false);
  const [f, setF] = useState({
    accountId: "", symbol: "XAUUSD", side: "BUY", status: "CLOSED",
    openedAt: new Date().toISOString().slice(0, 16),
    closedAt: new Date().toISOString().slice(0, 16),
    volume: "0.10", entryPrice: "0", exitPrice: "", grossPnl: "",
    commission: "0", swap: "0", session: "LONDON", timeframe: "H1",
    setupTag: "", notes: "",
  });

  useEffect(() => { fetch("/api/accounts").then((r) => r.json()).then((a) => {
    setAccounts(a); if (a[0]) setF((p) => ({ ...p, accountId: a[0].id }));
  }); }, []);

  async function submit(e: React.FormEvent) {
    e.preventDefault(); setBusy(true); setErr(null);
    const payload = { ...f,
      openedAt: new Date(f.openedAt).toISOString(),
      closedAt: f.closedAt ? new Date(f.closedAt).toISOString() : null,
      exitPrice: f.exitPrice || null,
      grossPnl: f.grossPnl || null,
    };
    const res = await fetch("/api/trades", { method: "POST", headers: { "content-type": "application/json" }, body: JSON.stringify(payload) });
    setBusy(false);
    if (!res.ok) { const j = await res.json().catch(() => ({})); setErr(j.error ?? "Failed"); return; }
    r.push("/trades");
  }

  const set = (k: string) => (e: any) => setF({ ...f, [k]: e.target.value });

  return (<><Nav /><main className="max-w-2xl mx-auto p-6">
    <h1 className="text-2xl font-semibold mb-4">New trade</h1>
    <form onSubmit={submit} className="grid grid-cols-2 gap-2">
      <select value={f.accountId} onChange={set("accountId")} required>
        {accounts.map((a) => <option key={a.id} value={a.id}>{a.name}</option>)}
      </select>
      <input placeholder="Symbol" value={f.symbol} onChange={set("symbol")} required />
      <select value={f.side} onChange={set("side")}><option>BUY</option><option>SELL</option></select>
      <input placeholder="Volume" value={f.volume} onChange={set("volume")} required />
      <input placeholder="Entry price" value={f.entryPrice} onChange={set("entryPrice")} required />
      <input placeholder="Exit price" value={f.exitPrice} onChange={set("exitPrice")} />
      <input placeholder="Gross P&L" value={f.grossPnl} onChange={set("grossPnl")} />
      <input placeholder="Commission" value={f.commission} onChange={set("commission")} />
      <input placeholder="Swap" value={f.swap} onChange={set("swap")} />
      <input type="datetime-local" value={f.openedAt} onChange={set("openedAt")} required />
      <input type="datetime-local" value={f.closedAt} onChange={set("closedAt")} />
      <input placeholder="Session" value={f.session} onChange={set("session")} />
      <input placeholder="Timeframe" value={f.timeframe} onChange={set("timeframe")} />
      <input placeholder="Setup tag" value={f.setupTag} onChange={set("setupTag")} />
      <textarea className="col-span-2" placeholder="Notes" value={f.notes} onChange={set("notes")} rows={3} />
      {err && <div className="col-span-2 text-neg text-sm">{err}</div>}
      <button disabled={busy} className="col-span-2 bg-pos text-black rounded-md py-2 font-medium">{busy ? "Saving…" : "Save trade"}</button>
    </form>
  </main></>);
}
"use client";
import { useEffect, useState } from "react";
import { Nav } from "@/components/Nav";

export default function Import() {
  const [accounts, setAccounts] = useState<any[]>([]);
  const [accountId, setAccountId] = useState("");
  const [file, setFile] = useState<File | null>(null);
  const [result, setResult] = useState<any>(null);
  const [err, setErr] = useState<string | null>(null);
  const [busy, setBusy] = useState(false);

  useEffect(() => { fetch("/api/accounts").then((r) => r.json()).then((a) => { setAccounts(a); if (a[0]) setAccountId(a[0].id); }); }, []);

  async function submit(e: React.FormEvent) {
    e.preventDefault(); if (!file || !accountId) return;
    setBusy(true); setErr(null); setResult(null);
    const fd = new FormData(); fd.append("file", file); fd.append("accountId", accountId);
    const res = await fetch("/api/import/csv", { method: "POST", body: fd });
    setBusy(false);
    const j = await res.json();
    if (!res.ok) { setErr(j.error ?? "Import failed"); return; }
    setResult(j);
  }

  return (<><Nav /><main className="max-w-2xl mx-auto p-6">
    <h1 className="text-2xl font-semibold mb-4">Import CSV statement</h1>
    <form onSubmit={submit} className="flex flex-col gap-3">
      <select value={accountId} onChange={(e) => setAccountId(e.target.value)}>
        {accounts.map((a) => <option key={a.id} value={a.id}>{a.name}</option>)}
      </select>
      <input type="file" accept=".csv,text/csv" onChange={(e) => setFile(e.target.files?.[0] ?? null)} required />
      <button disabled={busy || !file} className="bg-pos text-black rounded-md py-2 font-medium">{busy ? "Importing…" : "Import"}</button>
      {err && <div className="text-neg text-sm">{err}</div>}
      {result && (
        <div className="border border-border rounded-lg p-4 text-sm">
          <div>Imported: <span className="text-pos">{result.imported}</span></div>
          <div>Skipped: <span className="text-warn">{result.skipped}</span></div>
          {result.errors?.length > 0 && (
            <details className="mt-2"><summary className="text-muted cursor-pointer">Errors</summary>
              <ul className="mt-1 list-disc list-inside text-muted text-xs">{result.errors.map((e: string, i: number) => <li key={i}>{e}</li>)}</ul>
            </details>
          )}
        </div>
      )}
    </form>
  </main></>);
}
# 1) Install
npm install
# 2) Configure
cp .env.example .env    # fill NEXTAUTH_SECRET via: openssl rand -base64 32
# 3) Start Postgres (example using Docker)
docker run -d --name nuel-pg -e POSTGRES_PASSWORD=postgres -e POSTGRES_DB=nueltrades -p 5432:5432 postgres:16
# 4) Create tables
npx prisma db push
# 5) Run
npm run dev
# 6) Tests
npm test