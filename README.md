# CtrlSync CRM v2 — SaaS Edition

Volledig werkende CRM SaaS applicatie. Responsive, dark/light mode, account systeem, team uitnodigingen, webhook integraties.

## Bestanden

| Bestand | Omschrijving |
|---|---|
| `index.html` | Landingspagina met pricing |
| `login.html` | Login pagina |
| `register.html` | Registratie (4 stappen: account → plan → team → klaar) |
| `app.html` | Volledig CRM dashboard |
| `styles.css` | Gedeelde design tokens & componenten |

## Functies

- ✅ Dark/Light mode (persistent)
- ✅ Responsive voor mobiel, tablet, desktop
- ✅ Account aanmaken + login
- ✅ 3 plannen: Solo €29/mo · Team €79/mo · Agency €199/mo
- ✅ Team uitnodigen via email
- ✅ Pipeline kanban board
- ✅ Leads & klanten tabel met zoekfunctie
- ✅ Deal tracking (waarde, gefactureerd, open bedrag)
- ✅ Takenbeheer gekoppeld aan leads
- ✅ Activiteiten feed (Calendly, Make, Webhook, handmatig)
- ✅ Integraties pagina met webhook URL, Calendly koppeling, Make guide
- ✅ Webhook test tool (simuleert inkomend event + maakt lead aan)
- ✅ Instellingen met notificatie toggles

## Live zetten op Vercel (gratis, aanbevolen)

### Optie 1: Via Vercel Dashboard (makkelijkst)
1. Ga naar [vercel.com](https://vercel.com) en maak een account
2. Klik "Add New Project" → "Upload"
3. Upload de hele `ctrlsync-crm-v2` map
4. Klik "Deploy"
5. Klaar — live op `jouwproject.vercel.app`

### Optie 2: Via GitHub + Vercel (aanbevolen voor updates)
1. Zet de bestanden in een GitHub repository
2. Verbind de repo in Vercel
3. Elke push naar main deployt automatisch

### Optie 3: Netlify
1. Ga naar [netlify.com](https://netlify.com)
2. Sleep de map naar het dashboard
3. Klaar in 30 seconden

## Upgrade naar productie (Supabase backend)

De app slaat nu data op in localStorage. Voor echte multi-user sync:

### 1. Supabase setup
```bash
npm install @supabase/supabase-js
```

### 2. Database schema (SQL)
```sql
-- Users & workspaces
create table workspaces (
  id uuid primary key default gen_random_uuid(),
  name text not null,
  plan text default 'team',
  created_at timestamptz default now()
);

create table users (
  id uuid primary key references auth.users,
  name text,
  email text,
  workspace_id uuid references workspaces(id),
  role text default 'member',
  created_at timestamptz default now()
);

-- CRM
create table leads (
  id bigserial primary key,
  workspace_id uuid references workspaces(id),
  naam text not null,
  bedrijf text, email text, telefoon text,
  status text default '🆕 Nieuw',
  bron text, niche text, prioriteit text,
  deal_waarde numeric default 0,
  gefactureerd numeric default 0,
  open_bedrag numeric default 0,
  volgende_actie date,
  dienst text, website text, notities text,
  created_at timestamptz default now()
);

create table taken (
  id bigserial primary key,
  workspace_id uuid references workspaces(id),
  lead_id bigint references leads(id),
  taak text not null,
  type text, status text default '📥 To Do',
  prioriteit text, deadline date, notitie text,
  created_at timestamptz default now()
);

create table activity_log (
  id bigserial primary key,
  workspace_id uuid references workspaces(id),
  type text, title text, body text, tag text,
  created_at timestamptz default now()
);
```

### 3. Row Level Security
```sql
-- Elke workspace ziet alleen zijn eigen data
alter table leads enable row level security;
create policy "workspace_isolation" on leads
  using (workspace_id = (select workspace_id from users where id = auth.uid()));
```

### 4. Webhooks instellen (voor Calendly/Make)
```js
// Supabase Edge Function: /functions/v1/webhook
const { createClient } = require('@supabase/supabase-js');

export default async function handler(req) {
  const { naam, email, bron, notities } = await req.json();
  const supabase = createClient(SUPABASE_URL, SUPABASE_SERVICE_KEY);
  
  await supabase.from('leads').insert({
    naam, email, bron, notities,
    status: '🆕 Nieuw',
    workspace_id: req.headers['x-workspace-id']
  });
  
  return new Response('OK', { status: 200 });
}
```

## Pricing Stripe integratie
Gebruik [Stripe Checkout](https://stripe.com/docs/checkout) voor betalingen:
- Solo: `price_solo_monthly`
- Team: `price_team_monthly`  
- Agency: `price_agency_monthly`

## Roadmap
- [ ] Supabase integratie
- [ ] Stripe betalingen
- [ ] Echte email uitnodigingen (via Resend/Postmark)
- [ ] Calendly OAuth koppeling
- [ ] PDF facturen
- [ ] CSV export
- [ ] Mobile PWA
- [ ] Klant portals (Agency plan)
