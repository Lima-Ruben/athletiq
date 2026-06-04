# AthletIQ — Complete Product & Engineering Documentation
## Master Reference Document v3.0

> **Stack:** Laravel · PostgreSQL · Vue.js + Inertia.js · Redis · Docker · Laravel Reverb
> **Target Users:** Track & Field Athletes, Coaches, Teams & Clubs, National Teams
> **Document Status:** Active — All product decisions locked
> **Purpose:** Single source of truth for product vision, business logic, architecture, and implementation

---

## Table of Contents

1. [Product Vision & Strategy](#1-product-vision--strategy)
2. [Confirmed Product Decisions](#2-confirmed-product-decisions)
3. [User Roles & Permission Model](#3-user-roles--permission-model)
4. [Architecture Patterns & Principles](#4-architecture-patterns--principles)
5. [Database Schema](#5-database-schema)
6. [Database Dictionary](#6-database-dictionary)
7. [Business Logic & Core Flows](#7-business-logic--core-flows)
8. [Training Load & Analytics Engine](#8-training-load--analytics-engine)
9. [Backend Folder Structure](#9-backend-folder-structure)
10. [Frontend Component Structure](#10-frontend-component-structure)
11. [API Design & Full Route Map](#11-api-design--full-route-map)
12. [API Reference (Key Endpoints)](#12-api-reference-key-endpoints)
13. [SaaS & Billing Architecture](#13-saas--billing-architecture)
14. [Real-Time Systems](#14-real-time-systems)
15. [Feature Roadmap — V1 / V2 / V3](#15-feature-roadmap--v1--v2--v3)
16. [Security Best Practices](#16-security-best-practices)
17. [Testing Strategy & BDD Specifications](#17-testing-strategy--bdd-specifications)
18. [Deployment Strategy](#18-deployment-strategy)
19. [UX/UI Design Direction](#19-uxui-design-direction)
20. [Difficult Engineering Challenges](#20-difficult-engineering-challenges)
21. [Business & Growth Considerations](#21-business--growth-considerations)

---

## 1. Product Vision & Strategy

### 1.1 The Problem

Track & field performance management is deeply fragmented. Athletes log workouts in spreadsheets, coaches manage feedback over WhatsApp, clubs use separate tools for scheduling, and nobody has a single source of truth for performance trends over a season. There is no serious, purpose-built platform that combines training management, performance analytics, coach collaboration, and club operations in one place for this sport.

### 1.2 The Product

**AthletIQ** is a centralized performance platform for track & field athletes, coaches, and clubs. It sits at the intersection of:

- **TrainingPeaks** — structured training load, periodization, compliance tracking
- **Linear** — dense, fast, keyboard-driven UX built for power users
- **Notion** — flexible, role-aware views that adapt to athlete vs coach vs admin context
- **Strava** — milestone celebration, personal records, seasonal progression

It is not a general fitness app. It is purpose-built for performance athletes who compete seriously and coaches who manage them professionally.

### 1.3 Core Value Propositions

**For Athletes:**
- One dashboard for all training data — sessions, load, recovery, PRs, competitions
- Readiness score every morning based on sleep, wellness, and training load
- Personal record tracking exclusively from official competition results
- Training plan compliance view — planned vs actual
- Competition history with split analysis

**For Coaches:**
- Real-time visibility across all assigned athletes
- Session review and structured feedback workflow
- Smart alerts for fatigue spikes and injury risk
- Custom training plan builder with athlete assignment
- Unplanned session detection and notifications

**For Clubs:**
- Centralized attendance recording and tracking
- Competition creation and athlete registration management
- Squad analytics and comparison dashboards
- Multi-coach, multi-squad management
- End-of-season performance reports

### 1.4 User Personas

**Marcus, 22 — Elite Sprinter (Club Athlete)**
Trains 6×/week under a club coach. Logs every sprint session in AthletIQ immediately after training. Cares deeply about his 100m PB and wants to understand whether his gym work correlates with race improvement over a season. Reviews his readiness score every morning. Competes nationally 4–6 times per season.

**Coach Sandra, 41 — Head Sprint Coach (Club Coach + Admin)**
Manages 18 athletes across two squads. Reviews all sessions every morning before training begins. Wants to see fatigue alerts before she pushes athletes through a hard session. Builds 8-week training plans and monitors compliance weekly. Also the club admin — she creates competitions, registers athletes, and manages the squad roster.

**James, 35 — National Team Coach**
Coaches the national 4×100m relay squad. His athletes all have individual club coaches. He uses AthletIQ to monitor training and recovery data during national camps and uses convocation access to evaluate athletes across different clubs for selection decisions.

**The Lions Athletics Club**
A club with 60 members, 4 coaches, and 3 squads. The club director (also head coach) uses AthletIQ to manage attendance, competition registrations, and generate end-of-season reports for federation funding submissions.

---

## 2. Confirmed Product Decisions

Every key product decision made during planning is documented here. These are locked and reflected throughout the rest of this document.

### 2.1 Organizations & Membership

| Decision | Answer |
|---|---|
| Can a coach belong to multiple organizations? | Yes — simultaneously |
| Can a coach have 1-on-1 athletes outside any org? | Yes — private coaching relationships independent of orgs |
| Can an athlete be in multiple organizations? | Yes, max 2 — one club + one national team. Never two clubs. |
| Can an athlete exist without any org? | Yes — independent self-tracking athletes are supported |
| Who invites coaches to an org? | The organization invites coaches — coaches do not self-assign |
| Who invites athletes to an org? | The organization invites athletes — or athletes join independently without an org |
| Can a head coach also be the club admin? | Yes — dual role is supported on a single account |
| Can coaches see each other's athletes inside the same org? | No — athletes are siloed per assigned coach |
| Is there a read-only staff role? | Yes — physio, manager, parent can be added as staff |
| What can staff see? | Only athletes assigned to them. Read-only. No write access to sessions or reviews. |

### 2.2 Data Ownership

| Decision | Answer |
|---|---|
| Who owns athlete data? | The athlete always owns their data |
| What happens when an athlete leaves an org? | The org loses access to that athlete's data going forward — only data from the membership period is retained in org reports |
| What happens when an org cancels its subscription? | Athlete data persists — athletes keep their accounts and all data regardless of org billing status |

### 2.3 Sessions & Training Plans

| Decision | Answer |
|---|---|
| Can coaches log sessions on behalf of athletes? | No — only athletes log their own sessions |
| Are planned and logged sessions distinct entities? | Yes — they are separate but can be linked via plan_session_id |
| What happens when an athlete logs a session not on their plan? | Detected as unplanned, flagged, and the coach is notified |
| What happens when an athlete has no active plan? | Sessions are logged freely with no unplanned flag triggered |
| Are training plans in scope? | Yes — V1 gets a simple list-based version, V2 gets full drag-and-drop builder |

### 2.4 Competitions & Personal Records

| Decision | Answer |
|---|---|
| Who creates competitions? | Organization admins and coaches create competitions |
| Who registers athletes for competitions? | Coaches register their assigned athletes |
| Are competitions shared/public records? | Yes — one competition record, multiple athletes from different orgs attach results |
| Can a PB come from a training session? | No — PBs are from official competition results only |
| What happens to a PB if its source result is deleted? | The PB is deleted automatically. Athlete is notified to resubmit if the deletion was incorrect. |

### 2.5 Communication & Notifications

| Decision | Answer |
|---|---|
| Can coaches broadcast messages to entire squads? | Yes |
| Are athletes notified when sessions are reviewed? | Yes |
| Notification channels | In-app + email in V1 and V2, push notifications in V3 (mobile) |

### 2.6 National Team Special Rules

| Decision | Answer |
|---|---|
| How is a national team org created? | Same process as a club — different org type field |
| Can national team coaches see athletes from other clubs? | Yes — for convocation/selection purposes via can_view_for_convocation flag. Read-only. |
| Does a national team coach's access override club privacy? | Yes for convocation-flagged coaches only — they can view any athlete's profile, sessions, and performance data regardless of which club the athlete belongs to |

### 2.7 Billing

| Decision | Answer |
|---|---|
| Who pays? | Organizations pay for all members under them |
| Are coaches billable seats? | No — coaches are free seats, orgs pay per athlete count |
| Is there a free tier? | No — all users must belong to a paying org or have a paid individual plan |
| What happens to athlete data when an org cancels? | Athlete data persists. The org loses management features, not the athletes' personal data. |

### 2.8 Onboarding

All new users complete an onboarding wizard on first login:

1. **Step 1 — Events:** Set primary event (e.g. 100m), add secondary events
2. **Step 2 — Goals:** Set season goal (e.g. "Sub-10.80 by August"), target competition date
3. **Step 3 — Coach:** Enter coach's email to request connection (optional)
4. **Step 4 — Organization:** Accept pending org invite or skip (independent athlete)

The `onboarded_at` timestamp is set upon wizard completion. Users with `onboarded_at = NULL` are always redirected to the wizard on login.

---

## 3. User Roles & Permission Model

### 3.1 Role Hierarchy

The platform uses a two-level role system: a global system role and an organization-scoped membership role.

**Global roles (stored on the user record):**
- `superadmin` — Anthropic/system level access
- `user` — all regular users (role differentiated by context)

**Organization membership roles (stored on memberships table):**
- `admin` — org-level admin, full access including billing
- `coach` — manages assigned athletes, can create plans and competitions
- `staff` — read-only access to assigned athletes (physio, manager, parent)
- `athlete` — self-access only to own data

**Special flags:**
- `can_view_for_convocation` — on national team coach memberships only, grants cross-org read access

### 3.2 Permission Matrix

| Action | Org Admin | Coach | Staff | Athlete |
|---|---|---|---|---|
| Create organization | Admin only | — | — | — |
| Invite coaches to org | Yes | — | — | — |
| Invite athletes to org | Yes | Yes | — | — |
| Create / manage squads | Yes | — | — | — |
| Assign athletes to coach | Yes | — | — | — |
| View own assigned athletes | Yes | Yes (own only) | Yes (own only) | — |
| View all org athletes | Yes | — | — | — |
| Log training session | — | — | — | Yes (self only) |
| Review / approve sessions | Yes | Yes (own athletes) | — | — |
| Leave coach feedback | Yes | Yes (own athletes) | — | — |
| Create training plans | Yes | Yes | — | — |
| Assign plans to athletes | Yes | Yes (own athletes) | — | — |
| Create competitions | Yes | Yes | — | — |
| Register athletes for competitions | Yes | Yes (own athletes) | — | — |
| Log competition result | — | — | — | Yes (self only) |
| View squad analytics | Yes | Yes (own squad) | Yes (own squad, read) | — |
| Record attendance | Yes | Yes | — | — |
| Manage billing / subscription | Yes | — | — | — |
| National team cross-org view | Convocation flag only | Convocation flag only | — | — |

### 3.3 The Two-Organization Rule

An athlete can belong to a maximum of two organizations simultaneously:
- Maximum 1 organization of type `club`
- Maximum 1 organization of type `national_team`

Two clubs at the same time is not allowed. This rule is enforced at the invitation acceptance layer by the `OrgLimitEnforcer` service, and has database-level uniqueness on `(user_id, organization_id)` as an additional guard.

**Enforcement logic:**

```php
class OrgLimitEnforcer
{
    public function check(User $user, Organization $targetOrg): void
    {
        if ($targetOrg->type === 'club') {
            $alreadyInClub = $user->memberships()
                ->join('organizations', 'organizations.id', '=', 'memberships.organization_id')
                ->where('organizations.type', 'club')
                ->where('memberships.status', 'active')
                ->exists();

            throw_if($alreadyInClub, AlreadyBelongsToClubException::class,
                'You already belong to a club. Leave your current club first.');
        }

        if ($targetOrg->type === 'national_team') {
            $alreadyInNT = $user->memberships()
                ->join('organizations', 'organizations.id', '=', 'memberships.organization_id')
                ->where('organizations.type', 'national_team')
                ->where('memberships.status', 'active')
                ->exists();

            throw_if($alreadyInNT, AlreadyBelongsToNationalTeamException::class,
                'You already belong to a national team.');
        }
    }
}
```

### 3.4 National Team Convocation Access

National team coaches with `can_view_for_convocation = true` on their membership can access any athlete's data for selection purposes, regardless of which club that athlete belongs to. This is read-only and every access is logged in the audit trail.

```php
class AthletePolicy
{
    public function view(User $coach, AthleteProfile $athlete): bool
    {
        // Coach sees their own assigned athletes
        if (CoachAthleteAssignment::where('coach_id', $coach->id)
            ->where('athlete_id', $athlete->user_id)->exists()) {
            return true;
        }

        // National team coach with convocation rights sees any athlete
        return $coach->memberships()
            ->join('organizations', 'organizations.id', '=', 'memberships.organization_id')
            ->where('organizations.type', 'national_team')
            ->where('memberships.can_view_for_convocation', true)
            ->where('memberships.status', 'active')
            ->exists();
    }
}
```

### 3.5 Coach-Athlete Silo

Coaches inside the same organization cannot see each other's athletes. The `EnsureCoachOwnsAthlete` middleware validates every coach-to-athlete route against the `coach_athlete_assignments` table. There is no org-wide athlete feed for coaches — only their assigned roster.

---

## 4. Architecture Patterns & Principles

### 4.1 Domain-Driven Design Lite

Code is organized by business domain, not by Laravel's default MVC directory structure. Each domain is a self-contained module owning its models, actions, services, repositories, events, listeners, and DTOs.

**Domains:**
- `Training` — sessions, plans, plan assignments, sprint work, gym work, exercises
- `Performance` — personal records, competition results, PR detection
- `Recovery` — wellness logs, sleep logs, injury logs, readiness scores
- `Team` — organizations, squads, memberships, invitations, attendance, coach-athlete assignments
- `Identity` — users, athlete profiles, coach profiles, onboarding
- `Communication` — notifications, messages, threads, broadcasts, session reviews
- `Competition` — competitions, registrations, results
- `Analytics` — load queries, trend queries, compliance queries, dashboard aggregation
- `Billing` — subscriptions, plans, seat enforcement, Stripe integration

Cross-domain communication happens exclusively via Domain Events dispatched through Laravel's event system. Domains never import models from other domains directly.

### 4.2 CQRS-Inspired Separation

**Commands (writes):** Every mutation is an Action class with a single responsibility. Actions are the only entry point for state changes.

**Queries (reads):** Dedicated Query classes handle complex reads. These are allowed to use raw SQL or query builder for performance — they never go through Action classes.

**Events:** Domain events carry side effects (notifications, cache busting, analytics jobs, WebSocket broadcasts) away from the Action classes, keeping them pure.

### 4.3 Request Lifecycle

```
HTTP Request
  → Form Request (validation + authorization gate)
  → Controller (delegates only — zero business logic)
  → Action class (single use-case, orchestrates one operation)
  → Domain Service (complex multi-step business logic)
  → Repository (data access abstraction over Eloquent)
  → Eloquent Model (persistence layer)
  → Domain Event dispatched
      → Listeners
          → Notification dispatch jobs (default queue)
          → Analytics recalculation jobs (analytics queue)
          → Cache invalidation
          → WebSocket broadcasts (broadcasts queue)
```

### 4.4 Architectural Principles

1. **Thin controllers** — orchestrate only. Zero business logic inside controllers.
2. **One Action per use case** — `LogSessionAction`, `ApproveSessionAction`, `RegisterAthleteForCompetitionAction`. Never reuse an action for two purposes.
3. **Repositories for complex reads** — never put 4-join analytical queries in a controller or service.
4. **Events for side effects** — notifications, load recalculation, and cache invalidation are always triggered by events, never called directly from actions.
5. **Queue everything slow** — email, PDF generation, ACWR calculation, AI summaries = background jobs, always.
6. **Cache reads, invalidate on writes** — athlete dashboards and squad overviews are cached in Redis, invalidated by the relevant domain events.
7. **Policies for every authorization decision** — no inline `if ($user->role === 'coach')` checks anywhere.
8. **DTOs for inter-layer data transfer** — typed data objects between layers. Never raw arrays.
9. **Never expose sequential IDs** — UUIDs everywhere in public-facing routes and API responses.

---

## 5. Database Schema

### 5.1 Design Decisions

- **UUIDs** on all primary keys — prevents enumeration attacks, cleaner for distributed systems
- **JSONB** used precisely for flexible structured data: weather, splits, certifications, settings, action items
- **PostgreSQL arrays** for small infrequently-queried multi-value fields: event lists, tags, muscle groups
- **Soft deletes** on users and training sessions — athlete data is never permanently destroyed
- **Separate load metrics table** — `session_load_metrics` is always populated asynchronously, never blocking writes
- **Row-level multi-tenancy** — all org-scoped tables carry `organization_id`
- **Audit trail** — sensitive mutations logged to `audit_logs`
- **Cascade strategy** — `competition_results` cascade-deletes linked `personal_records` to maintain integrity

### 5.2 Full Schema

```sql
-- =============================================
-- IDENTITY & AUTH
-- =============================================

CREATE TABLE users (
    id                  UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    email               VARCHAR(255) UNIQUE NOT NULL,
    password_hash       VARCHAR(255) NOT NULL,
    first_name          VARCHAR(100) NOT NULL,
    last_name           VARCHAR(100) NOT NULL,
    avatar_url          VARCHAR(500),
    timezone            VARCHAR(100) DEFAULT 'UTC',
    locale              VARCHAR(10) DEFAULT 'en',
    two_factor_secret   VARCHAR(255),
    two_factor_enabled  BOOLEAN DEFAULT FALSE,
    onboarded_at        TIMESTAMP,
    last_active_at      TIMESTAMP,
    created_at          TIMESTAMP DEFAULT NOW(),
    updated_at          TIMESTAMP DEFAULT NOW(),
    deleted_at          TIMESTAMP
);

CREATE TABLE athlete_profiles (
    id                      UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id                 UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
    date_of_birth           DATE,
    gender                  VARCHAR(20),
    primary_event           VARCHAR(100),
    secondary_events        TEXT[],
    dominant_hand           VARCHAR(10),
    height_cm               DECIMAL(5,2),
    weight_kg               DECIMAL(5,2),
    bio                     TEXT,
    personal_coach_id       UUID REFERENCES users(id),
    active_plan_id          UUID,
    target_competition_date DATE,
    season_start_date       DATE,
    nationality             VARCHAR(100),
    athlete_number          VARCHAR(50),
    created_at              TIMESTAMP DEFAULT NOW(),
    updated_at              TIMESTAMP DEFAULT NOW()
);

CREATE TABLE coach_profiles (
    id                  UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id             UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
    bio                 TEXT,
    specializations     TEXT[],
    certifications      JSONB DEFAULT '[]',
    years_experience    INT,
    license_number      VARCHAR(100),
    created_at          TIMESTAMP DEFAULT NOW(),
    updated_at          TIMESTAMP DEFAULT NOW()
);

-- =============================================
-- ORGANIZATIONS
-- =============================================

CREATE TABLE organizations (
    id                      UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    name                    VARCHAR(255) NOT NULL,
    slug                    VARCHAR(100) UNIQUE NOT NULL,
    type                    VARCHAR(50) NOT NULL DEFAULT 'club',
    logo_url                VARCHAR(500),
    country                 VARCHAR(100),
    city                    VARCHAR(100),
    website                 VARCHAR(500),
    owner_id                UUID NOT NULL REFERENCES users(id),
    subscription_id         UUID,
    settings                JSONB DEFAULT '{}',
    created_at              TIMESTAMP DEFAULT NOW(),
    updated_at              TIMESTAMP DEFAULT NOW()
);

-- type values: club | national_team | school | professional

CREATE TABLE squads (
    id                  UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    organization_id     UUID NOT NULL REFERENCES organizations(id) ON DELETE CASCADE,
    name                VARCHAR(255) NOT NULL,
    event_group         VARCHAR(100),
    head_coach_id       UUID REFERENCES users(id),
    description         TEXT,
    created_at          TIMESTAMP DEFAULT NOW(),
    updated_at          TIMESTAMP DEFAULT NOW()
);

-- event_group values: sprints | middle_distance | long_distance | throws | jumps | hurdles | combined | multi

CREATE TABLE memberships (
    id                          UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id                     UUID NOT NULL REFERENCES users(id),
    organization_id             UUID NOT NULL REFERENCES organizations(id),
    squad_id                    UUID REFERENCES squads(id),
    role                        VARCHAR(50) NOT NULL,
    status                      VARCHAR(50) DEFAULT 'active',
    can_view_for_convocation    BOOLEAN DEFAULT FALSE,
    joined_at                   TIMESTAMP DEFAULT NOW(),
    left_at                     TIMESTAMP,
    invited_by                  UUID REFERENCES users(id),
    UNIQUE(user_id, organization_id)
);

-- role values: athlete | coach | staff | admin
-- status values: active | suspended | invited | left

CREATE TABLE coach_athlete_assignments (
    id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    coach_id        UUID NOT NULL REFERENCES users(id),
    athlete_id      UUID NOT NULL REFERENCES users(id),
    organization_id UUID REFERENCES organizations(id),
    assigned_at     TIMESTAMP DEFAULT NOW(),
    assigned_by     UUID REFERENCES users(id),
    UNIQUE(coach_id, athlete_id, organization_id)
);

-- organization_id = NULL means personal/private coaching outside any org

CREATE TABLE invitations (
    id                  UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    organization_id     UUID NOT NULL REFERENCES organizations(id),
    invited_by          UUID NOT NULL REFERENCES users(id),
    email               VARCHAR(255) NOT NULL,
    role                VARCHAR(50) NOT NULL,
    squad_id            UUID REFERENCES squads(id),
    token               VARCHAR(255) UNIQUE NOT NULL,
    status              VARCHAR(50) DEFAULT 'pending',
    expires_at          TIMESTAMP NOT NULL,
    accepted_at         TIMESTAMP,
    accepted_by         UUID REFERENCES users(id),
    created_at          TIMESTAMP DEFAULT NOW()
);

-- status values: pending | accepted | expired | revoked

-- =============================================
-- TRAINING PLANS
-- =============================================

CREATE TABLE training_plans (
    id                  UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    coach_id            UUID NOT NULL REFERENCES users(id),
    organization_id     UUID REFERENCES organizations(id),
    title               VARCHAR(255) NOT NULL,
    description         TEXT,
    phase               VARCHAR(100),
    duration_weeks      INT NOT NULL,
    is_template         BOOLEAN DEFAULT FALSE,
    visibility          VARCHAR(50) DEFAULT 'private',
    tags                TEXT[],
    created_at          TIMESTAMP DEFAULT NOW(),
    updated_at          TIMESTAMP DEFAULT NOW()
);

-- phase values: base | build | peak | competition | recovery | transition
-- visibility values: private | squad | org

CREATE TABLE training_plan_assignments (
    id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    plan_id         UUID NOT NULL REFERENCES training_plans(id),
    athlete_id      UUID NOT NULL REFERENCES users(id),
    start_date      DATE NOT NULL,
    end_date        DATE,
    status          VARCHAR(50) DEFAULT 'active',
    assigned_by     UUID REFERENCES users(id),
    created_at      TIMESTAMP DEFAULT NOW()
);

-- status values: active | completed | cancelled

CREATE TABLE training_plan_sessions (
    id                      UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    plan_id                 UUID NOT NULL REFERENCES training_plans(id),
    day_offset              INT NOT NULL,
    title                   VARCHAR(255) NOT NULL,
    session_type            VARCHAR(100) NOT NULL,
    description             TEXT,
    prescribed_rpe          INT CHECK (prescribed_rpe BETWEEN 1 AND 10),
    estimated_duration_min  INT,
    notes                   TEXT,
    sort_order              INT DEFAULT 0
);

-- =============================================
-- TRAINING SESSIONS (LOGGED)
-- =============================================

CREATE TABLE training_sessions (
    id                      UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    athlete_id              UUID NOT NULL REFERENCES users(id),
    coach_id                UUID REFERENCES users(id),
    plan_session_id         UUID REFERENCES training_plan_sessions(id),
    organization_id         UUID REFERENCES organizations(id),
    title                   VARCHAR(255) NOT NULL,
    session_type            VARCHAR(100) NOT NULL,
    session_date            DATE NOT NULL,
    start_time              TIME,
    end_time                TIME,
    duration_minutes        INT,
    location                VARCHAR(255),
    surface                 VARCHAR(100),
    weather                 JSONB,
    status                  VARCHAR(50) DEFAULT 'logged',
    rpe                     INT CHECK (rpe BETWEEN 1 AND 10),
    total_distance_m        DECIMAL(10,2),
    is_unplanned            BOOLEAN DEFAULT FALSE,
    unplanned_notified_at   TIMESTAMP,
    notes                   TEXT,
    internal_notes          TEXT,
    video_url               VARCHAR(500),
    created_at              TIMESTAMP DEFAULT NOW(),
    updated_at              TIMESTAMP DEFAULT NOW(),
    deleted_at              TIMESTAMP
);

-- session_type values: track | gym | cross_training | rest | recovery | competition
-- surface values: track | grass | road | treadmill | indoor | sand
-- status values: logged | reviewed | approved | flagged
-- weather JSONB structure: {temp_c, humidity, wind_ms, conditions}

CREATE TABLE session_load_metrics (
    session_id      UUID PRIMARY KEY REFERENCES training_sessions(id),
    session_load    DECIMAL(10,2),
    acute_load      DECIMAL(10,2),
    chronic_load    DECIMAL(10,2),
    acwr            DECIMAL(6,4),
    monotony        DECIMAL(6,4),
    strain          DECIMAL(10,2),
    calculated_at   TIMESTAMP DEFAULT NOW()
);

-- Always written by background job, never synchronously

-- =============================================
-- SPRINT WORK
-- =============================================

CREATE TABLE sprint_sessions (
    id                  UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    session_id          UUID NOT NULL REFERENCES training_sessions(id),
    event_type          VARCHAR(50) NOT NULL,
    total_reps          INT,
    total_distance_m    DECIMAL(10,2),
    rest_between_reps_s INT,
    created_at          TIMESTAMP DEFAULT NOW()
);

CREATE TABLE sprint_reps (
    id                  UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    sprint_session_id   UUID NOT NULL REFERENCES sprint_sessions(id),
    rep_number          INT NOT NULL,
    distance_m          DECIMAL(8,2),
    time_seconds        DECIMAL(8,3),
    wind_ms             DECIMAL(5,2),
    reaction_time_ms    INT,
    splits              JSONB,
    notes               TEXT
);

-- splits JSONB structure: {"10m": 1.81, "20m": 2.85, "40m": 4.49, "60m": 6.21}

-- =============================================
-- GYM WORK
-- =============================================

CREATE TABLE exercises (
    id                  UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    name                VARCHAR(255) NOT NULL,
    category            VARCHAR(100),
    muscle_groups       TEXT[],
    equipment           TEXT[],
    movement_pattern    VARCHAR(100),
    is_custom           BOOLEAN DEFAULT FALSE,
    created_by          UUID REFERENCES users(id),
    description         TEXT,
    video_url           VARCHAR(500),
    created_at          TIMESTAMP DEFAULT NOW()
);

-- category values: compound | isolation | plyometric | core | mobility | olympic
-- movement_pattern values: push | pull | hinge | squat | carry | rotation | jump

CREATE TABLE gym_sessions (
    id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    session_id      UUID NOT NULL REFERENCES training_sessions(id),
    gym_type        VARCHAR(100),
    created_at      TIMESTAMP DEFAULT NOW()
);

-- gym_type values: strength | power | hypertrophy | maintenance | testing | circuit

CREATE TABLE gym_sets (
    id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    gym_session_id  UUID NOT NULL REFERENCES gym_sessions(id),
    exercise_id     UUID NOT NULL REFERENCES exercises(id),
    set_number      INT NOT NULL,
    set_type        VARCHAR(50),
    weight_kg       DECIMAL(8,2),
    reps            INT,
    rpe             INT CHECK (rpe BETWEEN 1 AND 10),
    time_seconds    INT,
    distance_m      DECIMAL(8,2),
    notes           VARCHAR(500),
    sort_order      INT DEFAULT 0
);

-- set_type values: warm_up | working | drop | failure | amrap | pause

-- =============================================
-- COMPETITIONS
-- =============================================

CREATE TABLE competitions (
    id                  UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    created_by_org_id   UUID NOT NULL REFERENCES organizations(id),
    created_by_user_id  UUID NOT NULL REFERENCES users(id),
    name                VARCHAR(255) NOT NULL,
    competition_date    DATE NOT NULL,
    end_date            DATE,
    location            VARCHAR(255),
    country             VARCHAR(100),
    competition_type    VARCHAR(100),
    surface             VARCHAR(50),
    altitude_m          DECIMAL(7,2),
    organization_body   VARCHAR(255),
    is_public           BOOLEAN DEFAULT TRUE,
    notes               TEXT,
    created_at          TIMESTAMP DEFAULT NOW(),
    updated_at          TIMESTAMP DEFAULT NOW()
);

-- competition_type values: local | regional | national | international | championship | trials
-- surface values: outdoor | indoor

CREATE TABLE competition_registrations (
    id                  UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    competition_id      UUID NOT NULL REFERENCES competitions(id),
    athlete_id          UUID NOT NULL REFERENCES users(id),
    registered_by       UUID NOT NULL REFERENCES users(id),
    events              TEXT[] NOT NULL,
    status              VARCHAR(50) DEFAULT 'registered',
    notes               TEXT,
    registered_at       TIMESTAMP DEFAULT NOW(),
    UNIQUE(competition_id, athlete_id)
);

-- status values: registered | withdrawn | dns (did not start) | dnf (did not finish)

CREATE TABLE competition_results (
    id                  UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    competition_id      UUID NOT NULL REFERENCES competitions(id),
    athlete_id          UUID NOT NULL REFERENCES users(id),
    event               VARCHAR(100) NOT NULL,
    round               VARCHAR(50),
    lane                INT,
    result_value        DECIMAL(10,4) NOT NULL,
    result_unit         VARCHAR(20) NOT NULL,
    wind_ms             DECIMAL(5,2),
    position            INT,
    qualified           BOOLEAN,
    reaction_time_ms    INT,
    splits              JSONB,
    notes               TEXT,
    created_at          TIMESTAMP DEFAULT NOW()
);

-- round values: heat | semi | final | qualifier | time_trial
-- result_unit values: seconds | meters
-- For time events (100m, 400m, 110mH): result_value in seconds
-- For field events (Long Jump, Shot Put): result_value in meters

CREATE TABLE personal_records (
    id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    athlete_id      UUID NOT NULL REFERENCES users(id),
    event           VARCHAR(100) NOT NULL,
    category        VARCHAR(50),
    result_value    DECIMAL(10,4) NOT NULL,
    result_unit     VARCHAR(20) NOT NULL,
    wind_ms         DECIMAL(5,2),
    altitude_m      DECIMAL(7,2),
    achieved_at     DATE NOT NULL,
    competition_id  UUID NOT NULL REFERENCES competitions(id),
    result_id       UUID NOT NULL REFERENCES competition_results(id) ON DELETE CASCADE,
    is_official     BOOLEAN DEFAULT TRUE,
    notes           TEXT,
    created_at      TIMESTAMP DEFAULT NOW()
);

-- ON DELETE CASCADE on result_id: deleting a competition_result deletes its personal_record
-- category values: outdoor | indoor | road

-- =============================================
-- RECOVERY & WELLNESS
-- =============================================

CREATE TABLE recovery_logs (
    id                  UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    athlete_id          UUID NOT NULL REFERENCES users(id),
    log_date            DATE NOT NULL,
    fatigue_score       INT CHECK (fatigue_score BETWEEN 1 AND 10),
    soreness_score      INT CHECK (soreness_score BETWEEN 1 AND 10),
    mood_score          INT CHECK (mood_score BETWEEN 1 AND 10),
    stress_score        INT CHECK (stress_score BETWEEN 1 AND 10),
    motivation_score    INT CHECK (motivation_score BETWEEN 1 AND 10),
    overall_readiness   INT CHECK (overall_readiness BETWEEN 1 AND 10),
    notes               TEXT,
    created_at          TIMESTAMP DEFAULT NOW(),
    UNIQUE(athlete_id, log_date)
);

-- All scores: 1 = worst, 10 = best
-- fatigue: 1 = completely exhausted, 10 = completely fresh
-- soreness: 1 = extremely sore/painful, 10 = no soreness
-- mood: 1 = terrible, 10 = excellent
-- stress: 1 = extreme life stress, 10 = fully relaxed
-- motivation: 1 = no motivation to train, 10 = highly motivated

CREATE TABLE sleep_logs (
    id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    athlete_id      UUID NOT NULL REFERENCES users(id),
    sleep_date      DATE NOT NULL,
    bedtime         TIME,
    wake_time       TIME,
    duration_hours  DECIMAL(4,2),
    quality_score   INT CHECK (quality_score BETWEEN 1 AND 10),
    hrv_ms          INT,
    resting_hr      INT,
    notes           TEXT,
    created_at      TIMESTAMP DEFAULT NOW(),
    UNIQUE(athlete_id, sleep_date)
);

-- sleep_date = the night of (e.g. sleep_date = Monday = Monday night into Tuesday morning)
-- hrv_ms = Heart Rate Variability in milliseconds (imported from wearable in V3)

CREATE TABLE injury_logs (
    id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    athlete_id      UUID NOT NULL REFERENCES users(id),
    reported_by     UUID REFERENCES users(id),
    body_region     VARCHAR(100),
    body_side       VARCHAR(20),
    injury_type     VARCHAR(100),
    severity        VARCHAR(50),
    onset_date      DATE NOT NULL,
    return_date     DATE,
    status          VARCHAR(50) DEFAULT 'active',
    mechanism       TEXT,
    treatment_notes TEXT,
    cleared_by      UUID REFERENCES users(id),
    created_at      TIMESTAMP DEFAULT NOW(),
    updated_at      TIMESTAMP DEFAULT NOW()
);

-- body_region values: hamstring | quad | calf | achilles | knee | hip | glute | lower_back | shoulder | ankle | foot
-- body_side values: left | right | bilateral
-- injury_type values: strain | sprain | overuse | contusion | tendinopathy | stress_reaction | tear
-- severity values: minor | moderate | severe
-- status values: active | recovering | resolved

CREATE TABLE readiness_scores (
    id                  UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    athlete_id          UUID NOT NULL REFERENCES users(id),
    score_date          DATE NOT NULL,
    score               DECIMAL(5,2) NOT NULL,
    sleep_component     DECIMAL(5,2),
    recovery_component  DECIMAL(5,2),
    load_component      DECIMAL(5,2),
    calculated_at       TIMESTAMP DEFAULT NOW(),
    UNIQUE(athlete_id, score_date)
);

-- score range: 0–100
-- Calculated by background job after recovery_log and sleep_log are submitted
-- Also recalculated when ACWR changes significantly

-- =============================================
-- ATTENDANCE
-- =============================================

CREATE TABLE attendance_records (
    id                  UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    athlete_id          UUID NOT NULL REFERENCES users(id),
    organization_id     UUID NOT NULL REFERENCES organizations(id),
    squad_id            UUID REFERENCES squads(id),
    session_date        DATE NOT NULL,
    status              VARCHAR(50) NOT NULL,
    check_in_time       TIME,
    notes               TEXT,
    recorded_by         UUID REFERENCES users(id),
    created_at          TIMESTAMP DEFAULT NOW()
);

-- status values: present | absent | excused | late | partial

-- =============================================
-- COACH FEEDBACK & REVIEWS
-- =============================================

CREATE TABLE session_reviews (
    id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    session_id      UUID NOT NULL REFERENCES training_sessions(id),
    reviewer_id     UUID NOT NULL REFERENCES users(id),
    status          VARCHAR(50) DEFAULT 'reviewed',
    rating          INT CHECK (rating BETWEEN 1 AND 5),
    technique_note  TEXT,
    load_note       TEXT,
    general_note    TEXT,
    action_items    JSONB DEFAULT '[]',
    reviewed_at     TIMESTAMP DEFAULT NOW(),
    created_at      TIMESTAMP DEFAULT NOW()
);

-- status values: reviewed | approved | flagged
-- action_items JSONB structure: [{text, due_date, completed}]
-- One review per session — updated if coach revises

-- =============================================
-- MESSAGING
-- =============================================

CREATE TABLE message_threads (
    id                  UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    type                VARCHAR(50) DEFAULT 'direct',
    subject             VARCHAR(255),
    organization_id     UUID REFERENCES organizations(id),
    squad_id            UUID REFERENCES squads(id),
    created_by          UUID NOT NULL REFERENCES users(id),
    created_at          TIMESTAMP DEFAULT NOW()
);

-- type values: direct | group | squad_broadcast

CREATE TABLE message_thread_participants (
    thread_id       UUID NOT NULL REFERENCES message_threads(id),
    user_id         UUID NOT NULL REFERENCES users(id),
    last_read_at    TIMESTAMP,
    PRIMARY KEY (thread_id, user_id)
);

CREATE TABLE messages (
    id          UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    thread_id   UUID NOT NULL REFERENCES message_threads(id),
    sender_id   UUID NOT NULL REFERENCES users(id),
    body        TEXT NOT NULL,
    attachments JSONB DEFAULT '[]',
    is_pinned   BOOLEAN DEFAULT FALSE,
    edited_at   TIMESTAMP,
    created_at  TIMESTAMP DEFAULT NOW()
);

-- attachments JSONB structure: [{file_name, url, size_bytes, mime_type}]

-- =============================================
-- NOTIFICATIONS
-- =============================================

CREATE TABLE notifications (
    id          UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id     UUID NOT NULL REFERENCES users(id),
    type        VARCHAR(100) NOT NULL,
    title       VARCHAR(255) NOT NULL,
    body        TEXT,
    data        JSONB DEFAULT '{}',
    channel     VARCHAR(50) DEFAULT 'in_app',
    read_at     TIMESTAMP,
    action_url  VARCHAR(500),
    created_at  TIMESTAMP DEFAULT NOW()
);

-- type values: session_reviewed | session_approved | session_flagged | pr_set |
--              pr_deleted | fatigue_alert | unplanned_session | competition_registered |
--              invitation_received | injury_reported | plan_assigned | message_received
-- channel values: in_app | email | push (V3)

-- =============================================
-- FILES
-- =============================================

CREATE TABLE file_uploads (
    id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    owner_id        UUID NOT NULL REFERENCES users(id),
    organization_id UUID REFERENCES organizations(id),
    entity_type     VARCHAR(100),
    entity_id       UUID,
    file_name       VARCHAR(500) NOT NULL,
    storage_path    VARCHAR(1000) NOT NULL,
    mime_type       VARCHAR(100),
    size_bytes      BIGINT,
    is_public       BOOLEAN DEFAULT FALSE,
    created_at      TIMESTAMP DEFAULT NOW()
);

-- entity_type values: session | competition_result | profile | injury | plan

-- =============================================
-- BILLING
-- =============================================

CREATE TABLE subscription_plans (
    id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    name            VARCHAR(100) NOT NULL,
    slug            VARCHAR(100) UNIQUE NOT NULL,
    type            VARCHAR(50),
    price_monthly   DECIMAL(10,2),
    price_yearly    DECIMAL(10,2),
    max_athletes    INT,
    features        JSONB DEFAULT '{}',
    is_active       BOOLEAN DEFAULT TRUE,
    created_at      TIMESTAMP DEFAULT NOW()
);

-- type values: individual | club_starter | club_pro | enterprise
-- features JSONB example: {"ai_summaries": true, "api_access": false, "advanced_analytics": true}

CREATE TABLE subscriptions (
    id                          UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    subscriber_type             VARCHAR(50) NOT NULL,
    subscriber_id               UUID NOT NULL,
    plan_id                     UUID NOT NULL REFERENCES subscription_plans(id),
    status                      VARCHAR(50) DEFAULT 'trial',
    trial_ends_at               TIMESTAMP,
    current_period_start        TIMESTAMP,
    current_period_end          TIMESTAMP,
    stripe_subscription_id      VARCHAR(255),
    stripe_customer_id          VARCHAR(255),
    cancelled_at                TIMESTAMP,
    created_at                  TIMESTAMP DEFAULT NOW()
);

-- subscriber_type values: user | organization
-- subscriber_id is polymorphic: user.id for Athlete Pro, organization.id for club plans
-- status values: trial | active | past_due | cancelled | unpaid

-- =============================================
-- AUDIT LOG
-- =============================================

CREATE TABLE audit_logs (
    id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id         UUID REFERENCES users(id),
    organization_id UUID REFERENCES organizations(id),
    action          VARCHAR(255) NOT NULL,
    entity_type     VARCHAR(100),
    entity_id       UUID,
    old_values      JSONB,
    new_values      JSONB,
    ip_address      VARCHAR(45),
    user_agent      TEXT,
    created_at      TIMESTAMP DEFAULT NOW()
);

-- =============================================
-- PERFORMANCE INDEXES
-- =============================================

CREATE INDEX idx_users_email ON users(email);
CREATE INDEX idx_training_sessions_athlete_date ON training_sessions(athlete_id, session_date DESC);
CREATE INDEX idx_training_sessions_org ON training_sessions(organization_id, session_date DESC);
CREATE INDEX idx_training_sessions_status ON training_sessions(status) WHERE deleted_at IS NULL;
CREATE INDEX idx_training_sessions_unplanned ON training_sessions(athlete_id) WHERE is_unplanned = TRUE;
CREATE INDEX idx_sprint_reps_session ON sprint_reps(sprint_session_id);
CREATE INDEX idx_gym_sets_session ON gym_sets(gym_session_id);
CREATE INDEX idx_personal_records_athlete_event ON personal_records(athlete_id, event, achieved_at DESC);
CREATE INDEX idx_recovery_logs_athlete_date ON recovery_logs(athlete_id, log_date DESC);
CREATE INDEX idx_sleep_logs_athlete_date ON sleep_logs(athlete_id, sleep_date DESC);
CREATE INDEX idx_notifications_user_unread ON notifications(user_id, created_at DESC) WHERE read_at IS NULL;
CREATE INDEX idx_memberships_user ON memberships(user_id, status);
CREATE INDEX idx_memberships_org ON memberships(organization_id, role, status);
CREATE INDEX idx_coach_athlete_assignments ON coach_athlete_assignments(coach_id, athlete_id);
CREATE INDEX idx_competition_results_athlete ON competition_results(athlete_id);
CREATE INDEX idx_competition_registrations_comp ON competition_registrations(competition_id);
CREATE INDEX idx_session_load_metrics ON session_load_metrics(session_id);
CREATE INDEX idx_readiness_scores_athlete_date ON readiness_scores(athlete_id, score_date DESC);
CREATE INDEX idx_invitations_token ON invitations(token) WHERE status = 'pending';
CREATE INDEX idx_audit_logs_entity ON audit_logs(entity_type, entity_id, created_at DESC);
```

---

## 6. Database Dictionary

### `users`

| Column | Type | Nullable | Notes |
|---|---|---|---|
| id | UUID | NO | PK |
| email | VARCHAR(255) | NO | Unique. Used for login and all notifications. |
| password_hash | VARCHAR(255) | NO | Bcrypt. Never stored plain. |
| first_name | VARCHAR(100) | NO | — |
| last_name | VARCHAR(100) | NO | — |
| avatar_url | VARCHAR(500) | YES | S3 signed URL |
| timezone | VARCHAR(100) | NO | IANA string e.g. Europe/Lisbon. Used for date calculations. |
| locale | VARCHAR(10) | NO | e.g. en, pt, fr |
| two_factor_secret | VARCHAR(255) | YES | TOTP secret, encrypted at rest |
| two_factor_enabled | BOOLEAN | NO | Whether 2FA is active for this account |
| onboarded_at | TIMESTAMP | YES | NULL = must complete onboarding wizard on next login |
| last_active_at | TIMESTAMP | YES | Updated by TrackLastActivity middleware on each request |
| deleted_at | TIMESTAMP | YES | Soft delete. Data retained. Access revoked. |

---

### `memberships`

| Column | Type | Nullable | Notes |
|---|---|---|---|
| id | UUID | NO | PK |
| user_id | UUID | NO | FK → users |
| organization_id | UUID | NO | FK → organizations |
| squad_id | UUID | YES | FK → squads. Optional sub-group within org. |
| role | VARCHAR(50) | NO | athlete, coach, staff, or admin |
| status | VARCHAR(50) | NO | active, suspended, invited, or left |
| can_view_for_convocation | BOOLEAN | NO | True only for national team coaches. Grants cross-org read access. |
| joined_at | TIMESTAMP | NO | When membership became active |
| left_at | TIMESTAMP | YES | NULL = still active |
| invited_by | UUID | YES | FK → users — who sent the invite |

**Business Rules:**
- Unique constraint on (user_id, organization_id) — one membership record per user per org
- Two memberships with type=club is prevented at application layer
- can_view_for_convocation can only be true on national_team org memberships with role=coach

---

### `training_sessions`

| Column | Type | Nullable | Notes |
|---|---|---|---|
| athlete_id | UUID | NO | FK → users. Only athlete logs their own session. |
| coach_id | UUID | YES | FK → users. The athlete's assigned coach at time of logging. |
| plan_session_id | UUID | YES | FK → training_plan_sessions. Set when session is linked to a plan. |
| session_date | DATE | NO | The date the session happened — not the log date. |
| rpe | INT | YES | Rate of Perceived Exertion 1–10. Core input for load calculation. |
| duration_minutes | INT | YES | Used together with RPE for session load = RPE × duration. |
| is_unplanned | BOOLEAN | NO | True if athlete had active plan but logged with no plan_session_id |
| unplanned_notified_at | TIMESTAMP | YES | Timestamp of when the coach was notified. Prevents duplicate alerts. |
| status | VARCHAR(50) | NO | logged → reviewed → approved or flagged |
| internal_notes | TEXT | YES | Coach-only. Not visible to the athlete. |
| deleted_at | TIMESTAMP | YES | Soft delete. Triggers ACWR recalculation job when set. |

---

### `session_load_metrics`

| Column | Type | Nullable | Notes |
|---|---|---|---|
| session_id | UUID | NO | PK = FK → training_sessions. One row per session. |
| session_load | DECIMAL(10,2) | YES | RPE × duration_minutes (sRPE method) |
| acute_load | DECIMAL(10,2) | YES | 7-day rolling average load (ATL) |
| chronic_load | DECIMAL(10,2) | YES | 28-day rolling average load (CTL) |
| acwr | DECIMAL(6,4) | YES | ATL / CTL |
| monotony | DECIMAL(6,4) | YES | mean(daily_load) / std_dev(daily_load) |
| strain | DECIMAL(10,2) | YES | Weekly load × monotony |
| calculated_at | TIMESTAMP | NO | Timestamp of last background calculation |

**Business Rules:**
- Never populated synchronously — always written by RecalculateLoadMetricsJob
- Recalculated whenever any session in the athlete's last 28 days is created, updated, or soft-deleted
- ACWR > 1.5 triggers FatigueAlertTriggered event → coach notification

---

### `personal_records`

| Column | Type | Nullable | Notes |
|---|---|---|---|
| result_value | DECIMAL(10,4) | NO | Time in seconds for track events. Distance in meters for field events. |
| result_unit | VARCHAR(20) | NO | Always 'seconds' or 'meters' |
| competition_id | UUID | NO | Required. PRs are only from official competitions. |
| result_id | UUID | NO | FK → competition_results ON DELETE CASCADE. PR dies with its source result. |
| wind_ms | DECIMAL(5,2) | YES | Wind reading. Legal for records: ≤ +2.0 m/s for sprints/jumps. |

**Business Rules:**
- Only one active PB per athlete+event+category combination
- Created automatically by PRDetectionService when a competition result is logged
- Deleted automatically (cascade) when the source competition_result row is deleted
- Cannot be created from training session data

---

### `recovery_logs`

| Column | Type | Nullable | Notes |
|---|---|---|---|
| log_date | DATE | NO | Unique per athlete per day. Cannot submit twice for same day. |
| fatigue_score | INT | YES | 1=exhausted, 10=fresh. Inverted in readiness calculation. |
| soreness_score | INT | YES | 1=very sore, 10=no soreness. Inverted in readiness calculation. |
| mood_score | INT | YES | 1=terrible, 10=excellent |
| stress_score | INT | YES | Life stress (non-training). 1=extreme, 10=none. |
| motivation_score | INT | YES | 1=zero motivation, 10=highly motivated |
| overall_readiness | INT | YES | Self-reported overall feeling. Used in readiness score validation. |

---

### `competitions`

| Column | Type | Nullable | Notes |
|---|---|---|---|
| created_by_org_id | UUID | NO | Only org admins and coaches can create competitions |
| is_public | BOOLEAN | NO | Always true. Competitions are shared records any org can reference. |
| altitude_m | DECIMAL(7,2) | YES | Relevant for performance context in high-altitude venues |

**Business Rules:**
- Competitions are created by org admins/coaches — not athletes
- Athletes are registered by their coaches via competition_registrations
- Multiple athletes from different orgs can log results for the same competition
- Athletes log their own competition results — coaches do not log results on their behalf

---

### `subscriptions`

| Column | Type | Nullable | Notes |
|---|---|---|---|
| subscriber_type | VARCHAR(50) | NO | 'user' for Athlete Pro, 'organization' for club plans |
| subscriber_id | UUID | NO | Polymorphic. Points to user.id or organization.id. |
| status | VARCHAR(50) | NO | trial → active → past_due or cancelled |
| stripe_subscription_id | VARCHAR(255) | YES | Stripe's internal ID. Used for webhook reconciliation. |

**Business Rules:**
- Coaches are free seats — organizations pay per athlete count
- Athlete data persists after org cancels — data ownership belongs to athlete
- `EnforceSubscriptionLimits` middleware checks athlete count against plan max_athletes before creating new memberships

---

## 7. Business Logic & Core Flows

### 7.1 Athlete Joins an Organization

1. Org admin or coach sends an invitation: `POST /organizations/{id}/invite` with email, role, optional squad_id
2. System creates an invitation record with a unique secure token and expiry (7 days default)
3. Email dispatched to the invited address via queue
4. Athlete clicks the link → lands on `/invitations/{token}`
5. If not registered → shown registration form with invite context pre-filled
6. After login/register, `AcceptInvitationAction` runs:
   - Fetches invitation — validates token, status=pending, not expired
   - `OrgLimitEnforcer.check()` validates the two-org rule
   - If violated → 422 error with human-readable explanation
   - If valid → creates Membership record, updates invitation status to accepted
7. `MemberJoinedOrganization` event fires
8. Athlete is redirected to the org dashboard

### 7.2 Training Session → ACWR Recalculation

1. Athlete posts a new session: `POST /athletes/{id}/sessions`
2. `LogTrainingSessionAction` runs:
   - Validates athlete has access to log (themselves only)
   - Checks whether athlete has an active training plan assignment for today
   - If yes and no `plan_session_id` provided → sets `is_unplanned = true`
   - Creates TrainingSession record
3. `TrainingSessionLogged` event fires
4. Listeners run:
   - `NotifyCoachOfNewSession` → dispatches `SessionNotificationJob` to default queue
   - `RecalculateAthleteLoad` → dispatches `RecalculateLoadMetricsJob` to analytics queue
   - If `is_unplanned = true` → `NotifyCoachOfUnplannedSession` dispatches unplanned alert
5. `RecalculateLoadMetricsJob` executes:
   - Fetches all non-deleted sessions for athlete in last 28 days
   - Calculates session_load = RPE × duration_minutes
   - Calculates 7-day ATL (acute load), 28-day CTL (chronic load)
   - Calculates ACWR = ATL / CTL
   - Calculates monotony and strain
   - Upserts into `session_load_metrics`
   - If ACWR > 1.5 → fires `FatigueAlertTriggered` → coach receives red alert
   - Broadcasts `LoadUpdated` event to coach's private WebSocket channel
6. Coach dashboard updates in real-time via WebSocket

### 7.3 Competition Result → PR Detection

1. Athlete logs result: `POST /competitions/{id}/results`
2. System validates the athlete is registered for this competition via `competition_registrations`
3. `CompetitionResult` record is created
4. `CompetitionResultLogged` event fires
5. `PRDetectionService::checkForNewRecord($athlete, $event, $result)` runs:
   - Queries `personal_records` for athlete + event + category, ordered by result_value
   - For time events: lower result_value = better
   - For field events: higher result_value = better
   - If no existing PR → creates a new PersonalRecord
   - If existing PR and new result is better → creates new PersonalRecord (replaces current best)
6. If a new PR was created → `PersonalRecordSet` event fires:
   - Athlete receives notification: "🏆 New Personal Best — 100m: 10.74s"
   - Coach receives notification: "Marcus set a new 100m PB — 10.74s"
7. If source result is later deleted → cascade ON DELETE removes linked PersonalRecord automatically → `PersonalRecordDeleted` event fires → athlete notified to resubmit

### 7.4 Coach Registers Athlete for Competition

1. Coach creates competition: `POST /competitions` — org-scoped, visible to all org coaches
2. Coach registers athlete: `POST /competitions/{id}/register`
   - Policy checks coach is assigned to this athlete via `coach_athlete_assignments`
   - Creates `CompetitionRegistration` record with the events array
3. `CompetitionRegistered` event fires → athlete receives notification

### 7.5 Unplanned Session Detection

The system determines whether a session is unplanned by checking:

- Does this athlete have an active `training_plan_assignment` where today falls between `start_date` and `end_date`?
- If yes → does the logged session include a `plan_session_id`?
- If no `plan_session_id` was provided despite an active plan → `is_unplanned = true`
- A notification is sent to the coach: "Marcus logged an unplanned session — Sprint Drills"
- The notification also appears as a dual alert: both a push notification AND a session queue badge
- `unplanned_notified_at` is set to prevent duplicate notifications

**Important:** Athletes with no active plan can log sessions freely without any unplanned flag.

### 7.6 Session Review Workflow

```
Athlete logs session → status: 'logged'
Coach reviews session → status: 'reviewed' → athlete notified
Coach approves session → status: 'approved' → athlete notified
Coach flags session → status: 'flagged' → athlete notified with flag reason
```

A `session_review` record stores the coach's detailed feedback: technique notes, load notes, general notes, and action items. The coach can update the review after submission. The athlete can see all non-internal notes. `internal_notes` on the session itself are coach-only.

### 7.7 Readiness Score Calculation

Triggered after an athlete submits both their recovery log and sleep log for the day. If either is missing, the score is partial and flagged as incomplete.

Components and weights:
- Sleep quality (25%) — from sleep_logs.quality_score
- Sleep duration (15%) — normalized from sleep_logs.duration_hours (optimal = 8hrs)
- Fatigue (25%) — inverted from recovery_logs.fatigue_score (10 - fatigue)
- Soreness (15%) — inverted from recovery_logs.soreness_score (10 - soreness)
- Mood (10%) — from recovery_logs.mood_score
- Load balance (10%) — derived from current ACWR (score peaks at ACWR 1.0, drops at extremes)

Final score is scaled to 0–100. Stored in `readiness_scores` table with component breakdown.

**Color labels:**
- 80–100 → Green — Excellent
- 60–79 → Amber — Good
- 40–59 → Orange — Moderate
- 0–39 → Red — Poor

### 7.8 Invitation Expiry

A scheduled command `ExpireInvitations` runs daily and sets `status = 'expired'` on all invitations where `expires_at < NOW() AND status = 'pending'`. Expired invitation tokens return a 410 Gone response if visited.

### 7.9 Data Access After Membership Ends

When an athlete leaves an org (`left_at` is set):
- The org loses real-time access to the athlete's data
- Historical org reports that reference this athlete only include data from the period `joined_at` to `left_at`
- The athlete retains full access to all their own data — nothing is deleted
- This is enforced in all org-scoped analytics queries via a date-range filter on the membership period

---

## 8. Training Load & Analytics Engine

### 8.1 Core Calculations

**Session Load (sRPE method):**
```
session_load = RPE × duration_minutes
Example: RPE 8 × 75 minutes = 600 load units
```

**Acute Training Load (ATL) — 7-day rolling average:**
```
ATL = sum(session_loads for last 7 days) / 7
Represents short-term fatigue
```

**Chronic Training Load (CTL) — 28-day rolling average:**
```
CTL = sum(session_loads for last 28 days) / 28
Represents long-term fitness base
```

**Acute:Chronic Workload Ratio (ACWR):**
```
ACWR = ATL / CTL
```

| ACWR Range | Zone | Color | Meaning |
|---|---|---|---|
| < 0.8 | Under-training | Blue | Detraining risk. Not enough stress. |
| 0.8 – 1.3 | Optimal | Green | Sweet spot. Positive adaptation. |
| 1.3 – 1.5 | Caution | Amber | Monitor closely. Risk rising. |
| > 1.5 | Danger | Red | High injury risk. Coach alerted. |

**Training Monotony:**
```
Monotony = mean(daily_session_loads) / standard_deviation(daily_session_loads)
High monotony = same training daily = poor neural and physiological adaptation
```

**Training Strain:**
```
Strain = weekly_total_load × monotony
High strain combined with high ACWR = significant injury risk signal
```

### 8.2 Readiness Score Algorithm

```php
class ReadinessScoreCalculator
{
    public function calculate(RecoveryLog $recovery, SleepLog $sleep, LoadMetrics $load): float
    {
        $components = [
            'sleep_quality'  => $sleep->quality_score,
            'sleep_duration' => $this->normalizeSleep($sleep->duration_hours),
            'fatigue'        => 11 - $recovery->fatigue_score,   // inverted
            'soreness'       => 11 - $recovery->soreness_score,  // inverted
            'mood'           => $recovery->mood_score,
            'load_balance'   => $this->acwrToScore($load->acwr),
        ];

        $weights = [
            'sleep_quality'  => 0.25,
            'sleep_duration' => 0.15,
            'fatigue'        => 0.25,
            'soreness'       => 0.15,
            'mood'           => 0.10,
            'load_balance'   => 0.10,
        ];

        $raw = 0;
        foreach ($components as $key => $value) {
            $raw += $value * $weights[$key];
        }

        return round($raw * 10, 1); // scale 1–10 range to 0–100
    }

    private function normalizeSleep(float $hours): float
    {
        // Peak score at 8 hours, degrades symmetrically either side
        if ($hours >= 8) return min(10, $hours - 8 + 8);
        return max(1, $hours);
    }

    private function acwrToScore(float $acwr): float
    {
        // Peak score at ACWR 1.0, drops toward extremes
        if ($acwr >= 0.8 && $acwr <= 1.3) return 10;
        if ($acwr < 0.8) return max(1, $acwr * 10);
        if ($acwr > 1.3) return max(1, 10 - (($acwr - 1.3) * 15));
        return 5;
    }
}
```

### 8.3 Athlete Dashboard KPIs

| Metric | Source | Update Trigger |
|---|---|---|
| Readiness Score (0–100) | readiness_scores | After recovery + sleep log submitted |
| ACWR + Zone | session_load_metrics | After each session logged/deleted |
| Sessions this week vs target | training_sessions + plan | Live count |
| Days to next competition | competition_registrations + competitions | Daily |
| Days since last PR | personal_records | After competition result logged |
| Recovery check-in streak | recovery_logs | Daily |
| Weekly plan compliance % | training_plan_assignments + sessions | End of each day |

### 8.4 Coach Dashboard KPIs

| Metric | Source | Update Trigger |
|---|---|---|
| Athletes in red zone (ACWR > 1.5) | session_load_metrics | After each load recalculation |
| Athletes with poor readiness (< 40) | readiness_scores | After readiness calculated |
| Sessions pending review | training_sessions (status=logged) | After each session logged |
| Athletes without check-in today | recovery_logs | Daily |
| Squad average readiness | readiness_scores | After each readiness calc |
| Weekly compliance rate | plan assignments vs sessions | End of each day |

### 8.5 Analytics Caching Strategy

Expensive analytical queries are never run inline with web requests:

- **Athlete dashboard:** Cached in Redis with 5-minute TTL. Invalidated by `TrainingSessionLogged`, `RecoveryLogCreated`, `PersonalRecordSet` events.
- **Coach squad overview:** Cached in Redis with 10-minute TTL. Invalidated by any change to an athlete the coach owns.
- **Load chart data (12 weeks):** Pre-calculated nightly by scheduler, stored in Redis. Busted on session creation/deletion.
- **Season analytics:** Calculated on-demand for first request, cached 1 hour. No real-time requirement.

---

## 9. Backend Folder Structure

```
app/
├── Console/Commands/
│   ├── RecalculateLoadMetrics.php       -- Manual trigger for load recalc
│   ├── SendDailyAthleteDigests.php      -- Morning email with day summary
│   ├── ExpireInvitations.php            -- Nightly invitation cleanup
│   └── GenerateWeeklyInsights.php       -- Weekly AI insights prep (V3)

├── Domain/
│   ├── Training/
│   │   ├── Models/
│   │   │   ├── TrainingSession.php
│   │   │   ├── TrainingPlan.php
│   │   │   ├── TrainingPlanAssignment.php
│   │   │   ├── TrainingPlanSession.php
│   │   │   ├── SprintSession.php
│   │   │   ├── SprintRep.php
│   │   │   ├── GymSession.php
│   │   │   └── GymSet.php
│   │   ├── Actions/
│   │   │   ├── LogTrainingSessionAction.php
│   │   │   ├── CompleteSessionAction.php
│   │   │   ├── LinkSessionToPlanAction.php
│   │   │   ├── FlagUnplannedSessionAction.php
│   │   │   ├── AssignTrainingPlanAction.php
│   │   │   └── DuplicatePlanAction.php
│   │   ├── Services/
│   │   │   ├── ACWRCalculator.php
│   │   │   ├── TrainingLoadCalculator.php
│   │   │   ├── PlanComplianceTracker.php
│   │   │   └── UnplannedSessionDetector.php
│   │   ├── Repositories/
│   │   │   ├── TrainingSessionRepository.php
│   │   │   └── TrainingPlanRepository.php
│   │   ├── Events/
│   │   │   ├── TrainingSessionLogged.php
│   │   │   ├── TrainingSessionApproved.php
│   │   │   ├── TrainingSessionDeleted.php
│   │   │   ├── UnplannedSessionDetected.php
│   │   │   └── TrainingPlanAssigned.php
│   │   ├── Listeners/
│   │   │   ├── RecalculateAthleteLoad.php
│   │   │   ├── NotifyCoachOfNewSession.php
│   │   │   └── NotifyCoachOfUnplannedSession.php
│   │   └── DTOs/
│   │       ├── LogSessionData.php
│   │       └── SprintRepData.php

│   ├── Performance/
│   │   ├── Models/
│   │   │   ├── PersonalRecord.php
│   │   │   ├── Competition.php
│   │   │   ├── CompetitionRegistration.php
│   │   │   └── CompetitionResult.php
│   │   ├── Actions/
│   │   │   ├── LogCompetitionResultAction.php
│   │   │   ├── CreateCompetitionAction.php
│   │   │   ├── RegisterAthleteForCompetitionAction.php
│   │   │   └── WithdrawAthleteFromCompetitionAction.php
│   │   ├── Services/
│   │   │   ├── PRDetectionService.php
│   │   │   ├── PerformanceTrendAnalyzer.php
│   │   │   └── EventNormalizationService.php
│   │   ├── Events/
│   │   │   ├── PersonalRecordSet.php
│   │   │   ├── PersonalRecordDeleted.php
│   │   │   └── CompetitionResultLogged.php
│   │   └── Repositories/
│   │       └── PerformanceRepository.php

│   ├── Recovery/
│   │   ├── Models/
│   │   │   ├── RecoveryLog.php
│   │   │   ├── SleepLog.php
│   │   │   ├── InjuryLog.php
│   │   │   └── ReadinessScore.php
│   │   ├── Actions/
│   │   │   ├── LogRecoveryAction.php
│   │   │   ├── LogSleepAction.php
│   │   │   └── ReportInjuryAction.php
│   │   ├── Services/
│   │   │   ├── ReadinessScoreCalculator.php
│   │   │   └── FatigueRiskAnalyzer.php
│   │   └── Events/
│   │       ├── RecoveryLogSubmitted.php
│   │       └── FatigueAlertTriggered.php

│   ├── Team/
│   │   ├── Models/
│   │   │   ├── Organization.php
│   │   │   ├── Squad.php
│   │   │   ├── Membership.php
│   │   │   ├── CoachAthleteAssignment.php
│   │   │   └── Invitation.php
│   │   ├── Actions/
│   │   │   ├── InviteMemberAction.php
│   │   │   ├── AcceptInvitationAction.php
│   │   │   ├── AssignAthleteToCoachAction.php
│   │   │   ├── CreateSquadAction.php
│   │   │   └── RemoveMemberAction.php
│   │   ├── Services/
│   │   │   ├── MembershipService.php
│   │   │   ├── OrgLimitEnforcer.php
│   │   │   └── AttendanceTracker.php
│   │   └── Events/
│   │       ├── MemberJoinedOrganization.php
│   │       └── MemberLeftOrganization.php

│   ├── Identity/
│   │   ├── Models/
│   │   │   ├── User.php
│   │   │   ├── AthleteProfile.php
│   │   │   └── CoachProfile.php
│   │   ├── Actions/
│   │   │   ├── RegisterUserAction.php
│   │   │   ├── CompleteOnboardingAction.php
│   │   │   └── UpdateProfileAction.php
│   │   └── Services/
│   │       └── OnboardingService.php

│   ├── Communication/
│   │   ├── Models/
│   │   │   ├── Notification.php
│   │   │   ├── MessageThread.php
│   │   │   └── Message.php
│   │   ├── Actions/
│   │   │   ├── SendMessageAction.php
│   │   │   ├── BroadcastToSquadAction.php
│   │   │   └── MarkThreadReadAction.php
│   │   └── Notifications/
│   │       ├── SessionReviewedNotification.php
│   │       ├── PersonalRecordSetNotification.php
│   │       ├── FatigueAlertNotification.php
│   │       ├── UnplannedSessionNotification.php
│   │       ├── CompetitionRegisteredNotification.php
│   │       └── InvitationReceivedNotification.php

│   └── Analytics/
│       ├── Queries/
│       │   ├── AthleteLoadSummaryQuery.php
│       │   ├── SquadReadinessQuery.php
│       │   ├── PerformanceTrendQuery.php
│       │   ├── SeasonalProgressionQuery.php
│       │   └── ComplianceRateQuery.php
│       ├── Services/
│       │   ├── DashboardAggregator.php
│       │   └── InsightGenerator.php
│       └── Jobs/
│           ├── RecalculateLoadMetricsJob.php
│           ├── RecalculateReadinessScoreJob.php
│           └── GenerateWeeklyInsightsJob.php

├── Http/
│   ├── Controllers/
│   │   ├── Auth/
│   │   │   ├── LoginController.php
│   │   │   ├── RegisterController.php
│   │   │   └── OnboardingController.php
│   │   ├── Athlete/
│   │   │   ├── DashboardController.php
│   │   │   ├── SessionController.php
│   │   │   ├── RecoveryController.php
│   │   │   ├── SleepController.php
│   │   │   └── ProfileController.php
│   │   ├── Coach/
│   │   │   ├── AthleteOverviewController.php
│   │   │   ├── SessionReviewController.php
│   │   │   ├── TrainingPlanController.php
│   │   │   └── CompetitionController.php
│   │   ├── Team/
│   │   │   ├── OrganizationController.php
│   │   │   ├── SquadController.php
│   │   │   ├── MembershipController.php
│   │   │   ├── InvitationController.php
│   │   │   └── AttendanceController.php
│   │   └── Analytics/
│   │       ├── LoadManagementController.php
│   │       └── PerformanceTrendsController.php
│   ├── Middleware/
│   │   ├── EnsureOrganizationMember.php
│   │   ├── EnsureCoachOwnsAthlete.php
│   │   ├── EnforceSubscriptionLimits.php
│   │   ├── TrackLastActivity.php
│   │   └── ScopeToOrganization.php
│   └── Resources/
│       ├── Training/
│       │   ├── TrainingSessionResource.php
│       │   └── SessionLoadResource.php
│       ├── Performance/
│       │   ├── PersonalRecordResource.php
│       │   └── CompetitionResultResource.php
│       └── Athlete/
│           └── AthleteProfileResource.php

├── Jobs/
│   ├── RecalculateAthleteLoadMetrics.php
│   ├── RecalculateReadinessScore.php
│   ├── SendSessionDigestToCoach.php
│   ├── NotifyUnplannedSession.php
│   ├── GenerateAIPerformanceSummary.php    -- V3
│   └── ProcessAttendanceReport.php

└── Support/
    ├── Enums/
    │   ├── SessionType.php
    │   ├── OrganizationType.php
    │   ├── MembershipRole.php
    │   ├── AthleteEvent.php
    │   └── NotificationType.php
    └── ValueObjects/
        ├── Duration.php
        ├── PerformanceResult.php
        └── LoadMetrics.php
```

---

## 10. Frontend Component Structure

```
resources/js/
├── Pages/
│   ├── Auth/
│   │   ├── Login.vue
│   │   ├── Register.vue
│   │   └── Onboarding/
│   │       ├── Step1Events.vue
│   │       ├── Step2Goals.vue
│   │       ├── Step3Coach.vue
│   │       └── Step4Organization.vue
│   ├── Athlete/
│   │   ├── Dashboard.vue
│   │   ├── Training/
│   │   │   ├── SessionLog.vue
│   │   │   ├── SessionDetail.vue
│   │   │   ├── SessionHistory.vue
│   │   │   └── TrainingCalendar.vue
│   │   ├── Performance/
│   │   │   ├── PersonalBests.vue
│   │   │   ├── CompetitionHistory.vue
│   │   │   └── SprintAnalysis.vue
│   │   └── Recovery/
│   │       ├── RecoveryLog.vue
│   │       └── WellnessTrends.vue
│   ├── Coach/
│   │   ├── Dashboard.vue
│   │   ├── AthleteRoster.vue
│   │   ├── AthleteDetail.vue
│   │   ├── SessionQueue.vue
│   │   ├── PlanBuilder.vue
│   │   └── CompetitionManager.vue
│   ├── Team/
│   │   ├── Overview.vue
│   │   ├── Members.vue
│   │   ├── Squads.vue
│   │   ├── Attendance.vue
│   │   └── Settings.vue
│   └── Analytics/
│       ├── LoadManagement.vue
│       ├── PerformanceTrends.vue
│       └── SeasonalReport.vue

├── Components/
│   ├── ui/
│   │   ├── Button.vue
│   │   ├── Badge.vue
│   │   ├── Card.vue
│   │   ├── Modal.vue
│   │   ├── Sheet.vue             -- slide-over panel for quick actions
│   │   ├── Dropdown.vue
│   │   ├── Avatar.vue
│   │   ├── Tooltip.vue
│   │   ├── Skeleton.vue          -- loading state skeletons
│   │   └── Toast.vue
│   ├── charts/
│   │   ├── LoadChart.vue         -- ATL/CTL/ACWR line chart
│   │   ├── ACWRGauge.vue         -- radial gauge with zone coloring
│   │   ├── SprintProgressChart.vue
│   │   ├── RecoveryRadar.vue     -- radar chart for 5 wellness dimensions
│   │   ├── SeasonTimeline.vue
│   │   └── PerformanceTrendLine.vue
│   ├── training/
│   │   ├── SessionCard.vue
│   │   ├── SprintRepInput.vue
│   │   ├── GymSetRow.vue
│   │   ├── RPESelector.vue       -- 1–10 visual selector
│   │   └── SessionStatusBadge.vue
│   ├── athlete/
│   │   ├── AthleteCard.vue
│   │   ├── ReadinessScore.vue    -- animated color ring
│   │   ├── PRBadge.vue
│   │   └── FatigueAlert.vue
│   ├── coach/
│   │   ├── FeedbackComposer.vue
│   │   ├── SessionReviewPanel.vue
│   │   └── AthleteLoadGrid.vue   -- multi-athlete load overview table
│   └── layout/
│       ├── AppShell.vue
│       ├── Sidebar.vue
│       ├── TopNav.vue
│       ├── CommandPalette.vue    -- Cmd+K quick actions
│       └── NotificationCenter.vue

├── Composables/
│   ├── useTrainingSession.ts
│   ├── useAthleteLoad.ts
│   ├── useRecoveryLog.ts
│   ├── useNotifications.ts
│   ├── useRealtime.ts            -- WebSocket subscription manager
│   ├── usePermissions.ts         -- role-based UI rendering
│   └── useCommandPalette.ts

├── Stores/ (Pinia)
│   ├── auth.ts
│   ├── notifications.ts
│   ├── realtime.ts
│   └── ui.ts

└── Types/ (TypeScript)
    ├── athlete.ts
    ├── session.ts
    ├── performance.ts
    ├── recovery.ts
    └── organization.ts
```

---

## 11. API Design & Full Route Map

### 11.1 Conventions

- Base path: `/api/v1/`
- Authentication: Laravel Sanctum (SPA cookies) + Laravel Passport (third-party OAuth, V3)
- Response envelope: `{ data, meta, links }`
- Pagination: cursor-based for feeds, page-based for admin lists
- Errors: RFC 7807 Problem Details format
- Versioning: URL-based (`/api/v1/`, `/api/v2/`)
- All IDs in URLs and responses are UUIDs

### 11.2 Full Route Map

```
-- AUTH
POST   /api/v1/auth/register
POST   /api/v1/auth/login
POST   /api/v1/auth/logout
GET    /api/v1/auth/me
POST   /api/v1/auth/onboarding
POST   /api/v1/auth/2fa/enable
POST   /api/v1/auth/2fa/verify

-- ATHLETES
GET    /api/v1/athletes/{athlete}/dashboard
GET    /api/v1/athletes/{athlete}/sessions
POST   /api/v1/athletes/{athlete}/sessions
GET    /api/v1/athletes/{athlete}/sessions/{session}
PATCH  /api/v1/athletes/{athlete}/sessions/{session}
DELETE /api/v1/athletes/{athlete}/sessions/{session}

POST   /api/v1/athletes/{athlete}/sessions/{session}/sprint-reps
POST   /api/v1/athletes/{athlete}/sessions/{session}/sprint-reps/bulk
GET    /api/v1/athletes/{athlete}/sessions/{session}/sprint-reps

POST   /api/v1/athletes/{athlete}/sessions/{session}/gym-sets
POST   /api/v1/athletes/{athlete}/sessions/{session}/gym-sets/bulk
GET    /api/v1/athletes/{athlete}/sessions/{session}/gym-sets

GET    /api/v1/athletes/{athlete}/personal-records
GET    /api/v1/athletes/{athlete}/personal-records/{event}
GET    /api/v1/athletes/{athlete}/load
GET    /api/v1/athletes/{athlete}/readiness
GET    /api/v1/athletes/{athlete}/recovery
POST   /api/v1/athletes/{athlete}/recovery
GET    /api/v1/athletes/{athlete}/sleep
POST   /api/v1/athletes/{athlete}/sleep
GET    /api/v1/athletes/{athlete}/injuries
POST   /api/v1/athletes/{athlete}/injuries
PATCH  /api/v1/athletes/{athlete}/injuries/{injury}

-- COMPETITIONS
GET    /api/v1/competitions
POST   /api/v1/competitions                              -- org admin/coach only
GET    /api/v1/competitions/{competition}
PATCH  /api/v1/competitions/{competition}
POST   /api/v1/competitions/{competition}/register       -- coach registers athlete
DELETE /api/v1/competitions/{competition}/register/{athlete}
POST   /api/v1/competitions/{competition}/results        -- athlete logs own result
GET    /api/v1/competitions/{competition}/results

-- COACH
GET    /api/v1/coach/athletes                            -- all assigned athletes
GET    /api/v1/coach/session-queue                       -- sessions pending review
POST   /api/v1/coach/sessions/{session}/review
PATCH  /api/v1/coach/sessions/{session}/review
GET    /api/v1/coach/squad-overview                      -- multi-athlete status grid
GET    /api/v1/coach/alerts                              -- fatigue + injury alerts
GET    /api/v1/coach/compliance                          -- plan compliance rates

-- TRAINING PLANS
GET    /api/v1/plans
POST   /api/v1/plans
GET    /api/v1/plans/{plan}
PATCH  /api/v1/plans/{plan}
DELETE /api/v1/plans/{plan}
POST   /api/v1/plans/{plan}/sessions
PATCH  /api/v1/plans/{plan}/sessions/{planSession}
DELETE /api/v1/plans/{plan}/sessions/{planSession}
POST   /api/v1/plans/{plan}/assign

-- ORGANIZATIONS
GET    /api/v1/organizations/{org}
PATCH  /api/v1/organizations/{org}
GET    /api/v1/organizations/{org}/members
DELETE /api/v1/organizations/{org}/members/{user}
GET    /api/v1/organizations/{org}/squads
POST   /api/v1/organizations/{org}/squads
PATCH  /api/v1/organizations/{org}/squads/{squad}
GET    /api/v1/organizations/{org}/attendance
POST   /api/v1/organizations/{org}/attendance
POST   /api/v1/organizations/{org}/attendance/bulk
GET    /api/v1/organizations/{org}/analytics

-- INVITATIONS
POST   /api/v1/organizations/{org}/invite
GET    /api/v1/invitations/{token}
POST   /api/v1/invitations/{token}/accept
POST   /api/v1/invitations/{token}/decline

-- MESSAGING
GET    /api/v1/messages/threads
POST   /api/v1/messages/threads
GET    /api/v1/messages/threads/{thread}
POST   /api/v1/messages/threads/{thread}/messages
PATCH  /api/v1/messages/threads/{thread}/read
POST   /api/v1/messages/broadcast               -- squad broadcast

-- NOTIFICATIONS
GET    /api/v1/notifications
PATCH  /api/v1/notifications/{id}/read
POST   /api/v1/notifications/read-all

-- ANALYTICS
GET    /api/v1/analytics/load
GET    /api/v1/analytics/performance-trends
GET    /api/v1/analytics/recovery-trends
GET    /api/v1/analytics/seasonal-report
GET    /api/v1/analytics/squad-comparison
GET    /api/v1/analytics/compliance

-- BILLING
GET    /api/v1/billing/plans
GET    /api/v1/billing/subscription
POST   /api/v1/billing/subscribe
POST   /api/v1/billing/cancel
POST   /api/v1/billing/portal                   -- Stripe customer portal
```

---

## 12. API Reference (Key Endpoints)

### POST /api/v1/athletes/{athlete}/sessions

Log a new training session. Only callable by the athlete themselves.

**Request body:**
```json
{
  "title": "Sprint Speed Development",
  "session_type": "track",
  "session_date": "2025-11-01",
  "duration_minutes": 75,
  "rpe": 8,
  "location": "National Stadium Track",
  "surface": "track",
  "weather": {
    "temp_c": 18,
    "humidity": 62,
    "wind_ms": 1.2,
    "conditions": "sunny"
  },
  "notes": "Felt strong. Hit new flying 40m split.",
  "plan_session_id": "abc123-optional"
}
```

**Response 201:**
```json
{
  "data": {
    "id": "xyz789...",
    "title": "Sprint Speed Development",
    "session_type": "track",
    "session_date": "2025-11-01",
    "duration_minutes": 75,
    "rpe": 8,
    "status": "logged",
    "is_unplanned": false,
    "load_metrics": {
      "session_load": 600,
      "status": "calculating"
    }
  }
}
```

---

### POST /api/v1/athletes/{athlete}/sessions/{session}/sprint-reps/bulk

Add multiple sprint reps in a single request.

**Request body:**
```json
{
  "reps": [
    {
      "rep_number": 1,
      "distance_m": 60,
      "time_seconds": 6.82,
      "wind_ms": 0.4,
      "reaction_time_ms": 142,
      "splits": { "10m": 1.81, "20m": 2.88, "40m": 4.51 }
    },
    {
      "rep_number": 2,
      "distance_m": 60,
      "time_seconds": 6.79,
      "wind_ms": 0.6,
      "reaction_time_ms": 138,
      "splits": { "10m": 1.79, "20m": 2.85, "40m": 4.49 }
    }
  ]
}
```

---

### GET /api/v1/athletes/{athlete}/load

Current training load metrics for an athlete.

**Response 200:**
```json
{
  "data": {
    "athlete_id": "...",
    "current_acwr": 1.08,
    "acwr_zone": "optimal",
    "acute_load": 540,
    "chronic_load": 500,
    "monotony": 1.2,
    "strain": 3600,
    "last_calculated_at": "2025-11-01T20:15:00Z",
    "chart_data": [
      { "week": "2025-W42", "atl": 480, "ctl": 490, "acwr": 0.98 },
      { "week": "2025-W43", "atl": 540, "ctl": 500, "acwr": 1.08 }
    ]
  }
}
```

---

### POST /api/v1/competitions/{competition}/results

Athlete logs their own competition result.

**Request body:**
```json
{
  "event": "100m",
  "round": "final",
  "lane": 4,
  "result_value": 10.74,
  "result_unit": "seconds",
  "wind_ms": 0.8,
  "position": 2,
  "reaction_time_ms": 138,
  "splits": {
    "10m": 1.79,
    "20m": 2.85,
    "40m": 4.49,
    "60m": 6.21
  }
}
```

**Response 201:**
```json
{
  "data": {
    "result": { "id": "...", "event": "100m", "result_value": 10.74 },
    "personal_record": {
      "is_new_pb": true,
      "previous_pb": 10.78,
      "improvement": 0.04
    }
  }
}
```

---

### GET /api/v1/coach/squad-overview

Multi-athlete status grid for the coach dashboard.

**Response 200:**
```json
{
  "data": [
    {
      "athlete": {
        "id": "...",
        "name": "Marcus Johnson",
        "primary_event": "100m"
      },
      "readiness": { "score": 78, "label": "Good", "color": "green" },
      "load": { "acwr": 1.08, "zone": "optimal", "color": "green" },
      "last_session": {
        "date": "2025-11-01",
        "title": "Sprint Speed Development",
        "status": "reviewed"
      },
      "alerts": [],
      "pending_sessions": 0
    },
    {
      "athlete": { "name": "Tomás Ferreira", "primary_event": "200m" },
      "readiness": { "score": 45, "label": "Poor", "color": "red" },
      "load": { "acwr": 1.62, "zone": "danger", "color": "red" },
      "alerts": ["fatigue_spike", "load_danger"],
      "pending_sessions": 1
    }
  ]
}
```

---

### Standard Error Response (RFC 7807)

```json
{
  "type": "https://athletiq.app/errors/validation",
  "title": "Validation Failed",
  "status": 422,
  "detail": "The provided data is invalid.",
  "errors": {
    "rpe": ["The RPE must be between 1 and 10."],
    "session_date": ["The session date cannot be in the future."]
  }
}
```

---

## 13. SaaS & Billing Architecture

### 13.1 Pricing Tiers

| Plan | Monthly | Yearly | Max Athletes | Coaches | Target |
|---|---|---|---|---|---|
| **Athlete Pro** | €12 | €99 | 1 (self) | — | Independent competitor |
| **Club Starter** | €49 | €399 | 25 | Unlimited | Small clubs, school teams |
| **Club Pro** | €129 | €999 | 100 | Unlimited | Established athletics clubs |
| **Enterprise** | Custom | Custom | Unlimited | Unlimited | National federations, pro orgs |

Coaches are **free seats** on all plans. Plans pay per athlete count.

### 13.2 Feature Matrix

| Feature | Athlete Pro | Club Starter | Club Pro | Enterprise |
|---|---|---|---|---|
| Session logging | Yes | Yes | Yes | Yes |
| Sprint & gym tracking | Yes | Yes | Yes | Yes |
| Competition results & PRs | Yes | Yes | Yes | Yes |
| Recovery & wellness | Yes | Yes | Yes | Yes |
| Basic analytics | Yes | Yes | Yes | Yes |
| Training plans | — | Yes | Yes | Yes |
| Coach connection | Yes | Yes | Yes | Yes |
| Squad management | — | Yes | Yes | Yes |
| Attendance tracking | — | Yes | Yes | Yes |
| ACWR & load management | Yes | Yes | Yes | Yes |
| Advanced analytics | — | — | Yes | Yes |
| AI summaries | — | — | Yes | Yes |
| API access | — | — | Yes | Yes |
| White-label | — | — | — | Yes |
| SSO | — | — | — | Yes |
| Export & reports | — | Yes | Yes | Yes |

### 13.3 Subscription Enforcement

```php
class EnforceSubscriptionLimits
{
    public function handle(Request $request, Closure $next): Response
    {
        $org  = $request->route('organization');
        $plan = $org->subscription?->plan;

        if (!$plan || !$plan->is_active) {
            return response()->json([
                'type'   => 'https://athletiq.app/errors/subscription_required',
                'title'  => 'Active subscription required',
                'status' => 402,
            ], 402);
        }

        if ($plan->max_athletes !== null) {
            $currentCount = $org->memberships()
                ->where('role', 'athlete')
                ->where('status', 'active')
                ->count();

            if ($currentCount >= $plan->max_athletes) {
                return response()->json([
                    'type'        => 'https://athletiq.app/errors/athlete_limit_reached',
                    'title'       => 'Athlete limit reached',
                    'status'      => 402,
                    'detail'      => "Your plan allows {$plan->max_athletes} athletes.",
                    'upgrade_url' => route('billing.upgrade'),
                ], 402);
            }
        }

        return $next($request);
    }
}
```

### 13.4 Feature Flag Checks

```php
// In services and controllers — never in views
if (! $org->subscription->plan->hasFeature('ai_summaries')) {
    abort(402, 'Upgrade to Club Pro to access AI summaries.');
}

// SubscriptionPlan model
public function hasFeature(string $feature): bool
{
    return data_get($this->features, $feature, false) === true;
}
```

### 13.5 Billing Events from Stripe (Webhooks)

| Stripe Event | Action |
|---|---|
| `customer.subscription.created` | Activate subscription, set status=active |
| `customer.subscription.updated` | Update plan or period dates |
| `invoice.payment_succeeded` | Extend current_period_end |
| `invoice.payment_failed` | Set status=past_due, notify org admin |
| `customer.subscription.deleted` | Set status=cancelled, downgrade features |

---

## 14. Real-Time Systems

### 14.1 WebSocket Channel Strategy

```
Private channels (per authenticated user):
  private-user.{userId}             -- notifications, messages
  private-coach.{coachId}           -- athlete activity feed, alerts

Presence channels (multi-user awareness):
  presence-session.{sessionId}      -- live session (athlete + watching coach)
  presence-squad.{squadId}          -- squad training day

Events broadcast over WebSocket:
  LoadMetricsUpdated                -- to private-coach.{coachId}
  SessionLogged                     -- to private-coach.{coachId}
  SessionReviewed                   -- to private-user.{athleteId}
  FatigueAlertTriggered             -- to private-coach.{coachId}
  MessageReceived                   -- to private-user.{userId}
  PersonalRecordSet                 -- to private-user.{athleteId}
```

### 14.2 Queue Configuration

```
Queues (Redis-backed, processed by separate workers):
  critical   -- auth events, payment webhooks (processed immediately)
  default    -- notifications, session processing
  analytics  -- ACWR calculations, readiness scores, trend data
  heavy      -- PDF reports, AI summaries, bulk imports
  broadcasts -- WebSocket event broadcasts
  email      -- outbound mail

Worker processes:
  php artisan queue:work --queue=critical,default,analytics --tries=3
  php artisan queue:work --queue=heavy --timeout=120
  php artisan queue:work --queue=broadcasts,email
```

### 14.3 Scheduled Jobs

```
Daily at 07:00     -- SendDailyAthleteDigests (morning summary email)
Daily at 00:00     -- ExpireInvitations (cleanup)
Daily at 01:00     -- RecalculateAllReadinessScores (catch missed calculations)
Weekly on Monday   -- GenerateWeeklyInsights (V3)
```

---

## 15. Feature Roadmap — V1 / V2 / V3

### V1 — Foundation (Months 1–4)

Goal: Build the core daily loop that athletes actually use every day and coaches rely on for basic oversight.

**Authentication & Identity**
- Email/password registration with email verification
- Onboarding wizard (events, goals, coach email, org invite)
- Athlete and coach profiles with all core fields
- Role system: Athlete, Coach, Staff, OrgAdmin
- Organization creation
- Invitation system (email → token → accept flow)
- Squad creation and management
- Coach-athlete assignment
- Two-org rule enforcement

**Core Training Loop**
- Log training session (type, date, duration, RPE, location, weather, notes)
- Sprint session logging with reps (distance, time, wind, reaction, splits)
- Gym session logging (exercises from shared library, sets, weight, reps, RPE)
- Session history with filtering by type, date, status
- Session status workflow: logged → reviewed → approved / flagged

**Unplanned Session Detection**
- Active plan check on session log
- Unplanned flag and coach notification
- Dual channel notification (in-app + email)

**Performance Tracking**
- Competition creation by org admin/coach
- Athlete registration by coach
- Competition result logging by athlete
- PR auto-detection on result submission
- PR deletion cascade when result deleted
- PR history per event per athlete

**Recovery**
- Daily wellness check-in (fatigue, soreness, mood, stress, motivation)
- Sleep log entry (duration, quality, bedtime)
- Basic trend charts on athlete dashboard

**Training Plans — Simple Version**
- Coach creates plan as ordered session list
- Assign plan to athlete with start date
- Athlete views plan sessions on calendar
- Basic planned vs actual comparison

**Analytics — Basic**
- Training volume chart (weekly/monthly bar chart)
- PR progression per event (line chart)
- Session RPE trend over time
- Readiness score display (calculated from available data)

**Notifications**
- In-app notification center
- Email notifications for all key events
- Notification types: session reviewed, PR set, PR deleted, unplanned session, plan assigned, competition registered, invitation received

**Billing**
- Stripe integration
- All four pricing tiers seeded
- Subscription creation, upgrade, downgrade, cancellation
- Athlete seat limit enforcement
- Stripe webhook handling

**Attendance**
- Manual attendance recording by coach or admin
- Per-session date attendance entry
- Attendance history view

---

### V2 — Intelligence (Months 5–8)

Goal: Add the analytical layer that makes coaches make better decisions and athletes understand their training scientifically.

**Load Management**
- Automated ACWR calculation via background jobs
- ATL / CTL / ACWR visualization with zone coloring
- Training monotony and strain indicators
- Fatigue spike alerts (ACWR > 1.5 → red alert to coach)
- 12-week load chart with ATL/CTL/ACWR lines

**Readiness Score — Full Implementation**
- Composite 0–100 score from sleep + recovery + load
- Component breakdown visible to athlete
- Coach squad readiness grid (all athletes color-coded)
- Daily readiness trend chart

**Training Plans — Full Builder**
- Week-view drag-and-drop session organizer
- Phase tagging (base, build, peak, competition, recovery)
- Plan template library (coach can save and reuse)
- Plan compliance tracking dashboard
- Compliance rate per athlete per week

**Real-Time Features**
- WebSocket live session updates (coach sees athlete logging in real-time)
- In-app direct messaging (athlete ↔ coach)
- Squad broadcast messaging (coach → entire squad)
- Real-time notifications via WebSocket
- Online presence indicators on squad grid

**Injury Management**
- Injury reporting form with body region selection
- Severity classification
- Return-to-training timeline tracking
- Injury history per athlete
- Active injury indicator on coach squad grid

**Advanced Analytics**
- Performance trend analysis per event with regression line
- Seasonal periodization overview
- Squad comparison dashboard (all athletes side-by-side)
- Compliance rate analytics per athlete and squad
- CSV export for all data views
- PDF session report generation

**2FA**
- TOTP-based two-factor auth for coaches and admins

---

### V3 — Scale & Intelligence (Months 9–14)

Goal: Differentiate with AI, mobile, wearables, and an API platform.

**AI-Powered Features**
- Weekly AI performance summary per athlete (LLM-generated from training data)
- Anomaly detection for unusual load or wellness patterns
- Training recommendation engine
- Natural language session search

**Wearable Integration**
- Garmin, Polar, Apple Watch API sync
- HRV data import to sleep logs
- GPS session data import and visualization

**Mobile App**
- Native iOS and Android (React Native or Flutter)
- Full offline session logging with background sync
- Push notifications (third channel alongside in-app and email)
- Biometric authentication

**Public API**
- REST API with developer key management
- Webhook system for third-party integration
- Rate limiting and usage dashboards
- API documentation portal

**Business Intelligence**
- Club admin reports: attendance rates, retention, season summaries
- Season review PDF generation for funding applications
- National team selection reports
- Athlete data export

---

## 16. Security Best Practices

### 16.1 Authentication

- Laravel Sanctum for SPA authentication — cookie-based, CSRF protected
- Laravel Passport for public API OAuth 2.0 (V3)
- TOTP-based 2FA available for coaches and admins (V2)
- Account lockout after 5 failed login attempts via Redis-backed throttle
- Secure password requirements enforced at registration (min 8 chars, complexity)
- Password reset via time-limited signed URLs only

### 16.2 Authorization

- Policy class for every model — zero inline role checks anywhere in the codebase
- `EnsureCoachOwnsAthlete` middleware validates coach access on every coach→athlete route
- `ScopeToOrganization` middleware on all org-scoped routes enforces tenant isolation
- National team convocation access checked explicitly via `AthletePolicy::viewForConvocation()`
- Staff role enforced at policy level — all writes return 403 for staff users

### 16.3 Data Protection

- UUIDs on all public-facing IDs — no sequential ID enumeration possible
- Sensitive fields encrypted at rest using Laravel's encryption (injury treatment notes, medical data)
- File uploads: MIME type validation, max size limits, stored outside web root, served via signed S3 URLs with short TTL
- No PII in application logs — middleware strips sensitive fields before Laravel logs requests
- Athlete data access scoped strictly by membership period via date range queries
- `internal_notes` on sessions never returned in athlete-facing API responses

### 16.4 Infrastructure Security

- HTTPS enforced everywhere; HSTS header set with long max-age
- CORS configured strictly — only allowed frontend origins accepted
- Rate limiting on all API endpoints via Redis-backed throttle middleware
- Input validation via Form Request classes on every endpoint — `$request->all()` never used directly
- SQL injection not possible — Eloquent and query builder exclusively; all raw queries use parameterized bindings
- Audit log records all sensitive mutations: membership changes, data exports, billing actions, result deletions
- Stripe webhooks verified via signature validation before processing

---

## 17. Testing Strategy & BDD Specifications

### 17.1 Test Pyramid

**Unit Tests (60% of suite) — Fast, zero database:**
- `ACWRCalculator` — all load formula calculations
- `ReadinessScoreCalculator` — component weights and edge cases
- `PRDetectionService` — time events, field events, same value, null cases
- `OrgLimitEnforcer` — all two-org rule combinations
- Policy classes — permission resolution logic for each role

**Integration Tests (30%) — Real database, real queues:**
- `TrainingSessionLogged` event → full listener chain
- `CompetitionResult` → PR detection → notification
- `AcceptInvitationAction` → membership creation + limit check
- ACWR recalculation job output
- Cascade deletion of PersonalRecord when CompetitionResult deleted

**Feature Tests (10%) — Full HTTP stack:**
- Auth flows: register, login, onboarding wizard
- Athlete session CRUD end-to-end
- Coach can see own athletes, cannot see others' athletes
- Two-org rule enforced on invitation acceptance
- Subscription athlete limits enforced
- National team coach convocation access granted/denied correctly
- PR deleted when competition result deleted

### 17.2 BDD Specifications

```gherkin
Feature: Athlete Registration and Onboarding

  Scenario: New athlete registers and completes onboarding
    Given a new user visits /register
    When they fill in name, email, and password and submit
    Then their account is created and onboarded_at is NULL
    And they are redirected to the onboarding wizard
    When they complete all four steps of the wizard
    Then onboarded_at is set and they land on the athlete dashboard

  Scenario: Invitation-linked registration
    Given an org admin sent an invitation to new@athlete.com
    When new@athlete.com visits /invitations/{token}
    And they register a new account and accept the invitation
    Then a membership record is created with role athlete
    And they are redirected to the organization dashboard

---

Feature: Training Session Logging

  Scenario: Athlete logs a planned session
    Given an athlete has an active training plan
    And today's plan includes a sprint session
    When the athlete logs a session with that plan_session_id
    Then is_unplanned is FALSE
    And no unplanned coach alert is sent
    And the session appears completed on the athlete's calendar

  Scenario: Athlete logs a session outside their plan
    Given an athlete has an active training plan
    And today has no planned session
    When the athlete logs a session with no plan_session_id
    Then is_unplanned is TRUE
    And the coach receives: "Marcus logged an unplanned session — Sprint Drills"
    And the session shows an unplanned badge in the session history

  Scenario: Athlete with no plan logs freely
    Given an athlete has NO active training plan assignment
    When the athlete logs any session
    Then is_unplanned is FALSE and no coach alert is sent

  Scenario: Session logging triggers ACWR recalculation
    Given an athlete logs a session with RPE 8 and 75 minutes duration
    When the session is saved
    Then a RecalculateLoadMetricsJob is dispatched to the analytics queue
    And session_load = 600 is stored in session_load_metrics
    And the coach dashboard receives a real-time WebSocket update

  Scenario: ACWR spike triggers fatigue alert
    Given an athlete's chronic load (CTL) is 500
    When their acute load (ATL) rises to 800
    Then ACWR = 1.60 which exceeds 1.5
    And a FatigueAlertTriggered event fires
    And the coach receives a red alert notification

---

Feature: Competition Results and Personal Bests

  Scenario: Coach creates a competition
    Given an authenticated org coach
    When they POST to /competitions with valid data
    Then the competition is created and visible to all coaches in the org

  Scenario: Coach registers athlete for competition
    Given coach Sandra is assigned to athlete Marcus
    When Sandra registers Marcus for the 100m and 200m
    Then a CompetitionRegistration is created
    And Marcus receives: "Coach Sandra registered you for National Championships — 100m, 200m"

  Scenario: Athlete logs a result that is a new personal best
    Given Marcus has an existing 100m PB of 10.78 seconds
    When Marcus logs a competition result of 10.74 seconds
    Then PRDetectionService detects 10.74 < 10.78
    And a new PersonalRecord is created linked to the competition and result
    And Marcus receives: "New Personal Best — 100m: 10.74s"
    And Coach Sandra receives: "Marcus set a new 100m PB: 10.74s"

  Scenario: Competition result deletion removes the PB
    Given Marcus has a 100m PB of 10.74 from CompetitionResult ID:ABC
    When CompetitionResult ID:ABC is deleted
    Then the PersonalRecord for 10.74 is deleted via cascade
    And Marcus receives: "Your 100m PB (10.74s) was removed — resubmit if incorrect"

  Scenario: Training session time does not create a PB
    Given Marcus logs a sprint rep in training with a time of 10.60 seconds
    Then no PersonalRecord is created
    Because PBs are only from official competition results

---

Feature: Organization Membership and Two-Org Rule

  Scenario: Athlete joins a club
    Given athlete Marcus has no existing club membership
    When Marcus accepts an invitation from Lions AC (type: club)
    Then a membership record is created with status active
    And Marcus has access to club features

  Scenario: Athlete tries to join a second club — rejected
    Given athlete Marcus is already a member of Lions AC (type: club)
    When Marcus tries to accept an invitation from Tigers AC (also type: club)
    Then the invitation acceptance returns 422
    With message: "You already belong to a club. Leave your current club first."

  Scenario: Athlete joins national team while already in a club — allowed
    Given Marcus is a member of Lions AC (type: club)
    When Marcus accepts an invitation from Portugal National Team (type: national_team)
    Then the membership is created successfully
    And Marcus belongs to both organizations simultaneously

  Scenario: National team coach views club athlete data for convocation
    Given coach James has can_view_for_convocation = true on his national team membership
    And athlete Marcus belongs to Lions AC but NOT to James's national team
    When James accesses Marcus's performance data
    Then access is GRANTED as read-only
    And the access is recorded in audit_logs

  Scenario: Club coach cannot access another coach's athletes
    Given Sandra is assigned to Marcus and Pedro is assigned to Ana in the same club
    When Sandra tries to access Ana's training sessions
    Then the request returns 403 Forbidden

  Scenario: Staff member has read-only access to assigned athletes only
    Given a physio is added as staff assigned to Squad A
    When they view a Squad A athlete's recovery log
    Then access is GRANTED (read-only)
    When they attempt to approve a training session
    Then access is DENIED with 403

---

Feature: Billing and Subscription Limits

  Scenario: Org at athlete limit cannot add more athletes
    Given Lions AC is on the Club Starter plan (max 25 athletes)
    And the club currently has 25 active athlete members
    When the admin tries to send a new athlete invitation
    Then the request returns 402
    With message: "Athlete limit reached. Upgrade your plan."

  Scenario: Athlete data persists after org cancels subscription
    Given Lions AC cancels their subscription
    When athlete Marcus logs in
    Then Marcus can still access all his personal training data, PRs, and competition history
    And Marcus's data is not deleted or inaccessible
```

### 17.3 Tools

- **Pest PHP** — BDD-style expressive tests with Laravel integration
- **Larastan** (PHPStan level 8) — static analysis, strict type checking
- **Laravel Dusk** — browser tests for onboarding, invitation, and payment flows
- **Factories** with realistic seeded data for all models
- **GitHub Actions** — CI pipeline: lint → static analysis → full test suite → deploy to staging

---

## 18. Deployment Strategy

### 18.1 Docker Compose (Local Development)

```yaml
services:
  app:
    build: .
    volumes: [".:/var/www/html"]
    depends_on: [postgres, redis, reverb]

  worker:
    command: php artisan queue:work --queue=critical,default,analytics --tries=3
    depends_on: [postgres, redis]

  worker-heavy:
    command: php artisan queue:work --queue=heavy,email --timeout=120
    depends_on: [postgres, redis]

  scheduler:
    command: php artisan schedule:work

  reverb:
    command: php artisan reverb:start --port=8080

  postgres:
    image: postgres:16-alpine
    environment:
      POSTGRES_DB: athletiq
      POSTGRES_USER: athletiq
      POSTGRES_PASSWORD: secret
    volumes: ["pgdata:/var/lib/postgresql/data"]

  redis:
    image: redis:7-alpine

  minio:
    image: minio/minio
    command: server /data --console-address ":9001"
```

### 18.2 Production Architecture

```
Cloudflare (CDN + WAF + DDoS protection)
        ↓
Load Balancer (AWS ALB or DigitalOcean LB)
        ↓
App Servers × 2–4 (PHP-FPM + Nginx, auto-scaling)
        ↓
PostgreSQL Primary (writes) + Read Replica (analytics queries)
        ↓
Redis Cluster (cache + queues + sessions)
        ↓
S3 (file storage) + SES (email) + Reverb (WebSocket server)
        ↓
Stripe (billing webhooks)
```

Worker nodes are separate from app servers — analytics queue workers can scale independently without affecting web response times.

### 18.3 CI/CD Pipeline

```
Developer pushes to feature branch
  → PR opened
  → GitHub Actions triggers:
      1. PHP lint (Pint)
      2. Larastan static analysis (level 8)
      3. Pest test suite (unit + integration + feature)
  → Code review
  → Merge to main
  → Auto-deploy to staging (zero-downtime rolling deploy)
  → Smoke tests run against staging
  → Manual approval required for production
  → Deploy to production (zero-downtime)
  → Post-deploy smoke tests
  → Slack notification with deploy summary
```

### 18.4 Zero-Downtime Deployment

Use Laravel's built-in maintenance mode with `down --secret` to allow smoke testing during deploys. Deploy with rolling restarts on PHP-FPM so existing connections are not dropped. Database migrations run in a pre-deploy step — all migrations must be backwards-compatible with the previous release.

---

## 19. UX/UI Design Direction

### 19.1 Design Philosophy

**"A precision instrument, not a fitness app."**

Dark mode first — athletes log sessions at end of day; bright interfaces are fatiguing after training. Data-dense but never overwhelming — pack information thoughtfully, not carelessly. Every action must feel instant via optimistic UI updates. Keyboard-navigable throughout — coaches manage many athletes and should never be forced to reach for a mouse. Premium sports-brand energy: Whoop's confidence meets Linear's precision.

### 19.2 Athlete Dashboard Layout

```
Good morning, Marcus.     Sprint Speed Dev · Today

┌────────────┬────────────┬─────────────┬──────────────────┐
│ Readiness  │   ACWR     │ Wk Sessions │ Days to Comp     │
│  78 / 100  │  1.08 ✓   │   4 / 5     │    12 days       │
└────────────┴────────────┴─────────────┴──────────────────┘

Training Load — 12 weeks           [ATL ─── CTL ─── ACWR]
████████████████████░░░░░          [chart]

┌──────────────────────────┬───────────────────────────────┐
│ Recent Sessions           │ Personal Bests                 │
│ Mon  Sprint Dev   RPE 8   │ 100m   10.78   ↑ Aug 2025     │
│ Sat  Gym: Power   RPE 7   │ 200m   21.44   → Jun 2025     │
│ Thu  Tempo        RPE 6   │ LJ     7.32m   ↑ Sep 2025     │
└──────────────────────────┴───────────────────────────────┘
```

### 19.3 Coach Dashboard Layout

```
Sprint Squad Overview                          Today ▼

⚠ 2 athletes amber  ·  1 red  ·  5 sessions pending review

┌──────────┬───────────┬─────────┬──────────┬─────────────┐
│ Athlete  │ Readiness │  ACWR   │  Logged  │   Status    │
├──────────┼───────────┼─────────┼──────────┼─────────────┤
│ Marcus   │  78  ✓    │ 1.08 ✓  │ Today    │ ✓ reviewed  │
│ Keisha   │  61  ⚠    │ 1.41 ⚠  │ Today    │ Pending     │
│ Tomás    │  45  ✗    │ 1.62 ✗  │ Yesterday│ ⚠ Flagged   │
└──────────┴───────────┴─────────┴──────────┴─────────────┘
```

### 19.4 Key UX Principles

- **Command palette** (Cmd+K): log session, find athlete, jump to PR, compose message — accessible from anywhere
- **Optimistic UI**: all writes reflected immediately in the UI, rolled back on error with toast notification
- **Inline session logging**: bottom sheet or slide-over panel — no page navigation needed to log a workout
- **PR celebration**: subtle badge animation on new personal best — athletic precision, never gamified or childish
- **Readiness ring**: animated color-transitioning circular progress ring (green → amber → red)
- **ACWR gauge**: radial gauge animates from previous value on load, zone background color shifts
- **Attendance grid**: drag-select cells for bulk status marking — never click one-by-one
- **Unplanned session badge**: visually distinct treatment in session history and coach queue
- **Feedback composer**: supports markdown — coaches write structured, detailed notes

---

## 20. Difficult Engineering Challenges

### 20.1 ACWR Historical Recalculation Consistency

**Problem:** ACWR depends on 28 days of prior session data. Editing or soft-deleting a historical session invalidates all load calculations from that date forward for the athlete.

**Solution:** `RecalculateLoadMetricsJob` is idempotent — it always recomputes from the full 28-day window. Any session mutation (create, update, soft-delete) dispatches the job. The job fetches all non-deleted sessions in the window, recalculates all metrics, and upserts `session_load_metrics`. It is safe to run multiple times and safe to queue multiple instances since the last write wins and the calculation is deterministic.

### 20.2 The Two-Organization Rule with Race Conditions

**Problem:** If two org invitations arrive at the same time and the athlete accepts both simultaneously, the concurrent checks could both pass before either write completes.

**Solution:** Database-level uniqueness on `(user_id, organization_id)` ensures only one membership record per user per org. The application-level check in `OrgLimitEnforcer` runs inside a database transaction with a row-level lock on the user's existing memberships table. If the unique constraint throws on write, the second acceptance returns a graceful error.

### 20.3 Data Access Scoping After Membership Ends

**Problem:** Athlete leaves a club. Club admin should lose access to live data. But historical analytics reports that included that athlete during their membership period should still reflect accurate historical data.

**Solution:** All org-scoped analytics queries join against memberships with a date range filter:

```sql
WHERE memberships.user_id = athlete.id
  AND memberships.organization_id = :org_id
  AND training_sessions.session_date >= memberships.joined_at
  AND training_sessions.session_date <= COALESCE(memberships.left_at, NOW())
```

Athlete data is never deleted. Access is controlled purely by the membership's active date range.

### 20.4 National Team Cross-Organization Access Without Privilege Escalation

**Problem:** National team coaches need read access to any athlete's data for convocation. But this must not allow them to accidentally escalate to write access or to access athletes via unrelated routes.

**Solution:** Every access via convocation rights is gated by `AthletePolicy::viewForConvocation()` which checks for the `can_view_for_convocation` flag exclusively. The flag is never settable by coaches themselves — only org admins can set it via the membership management UI. All writes for that athlete go through normal coach ownership checks which convocation access never satisfies. Every convocation read is logged in `audit_logs`.

### 20.5 Personal Record Integrity on Cascaded Delete

**Problem:** A coach or athlete deletes a competition result that was the source of a personal record. The PB record must be cleaned up atomically.

**Solution:** `competition_results.id` is a foreign key on `personal_records.result_id` with `ON DELETE CASCADE`. The delete is atomic at the database level. The cascade happens inside the database transaction, so there is never a state where the result is gone but the PB record still exists. After deletion, the application layer fires a `PersonalRecordDeleted` domain event via a model observer on `CompetitionResult`, which notifies the athlete.

### 20.6 Unplanned Session Detection at Scale

**Problem:** For every session logged, the system must check whether the athlete has an active plan, and if so, whether today has a matching planned session. As the user base grows this check must remain fast.

**Solution:** Active plan assignment is cached per athlete in Redis with a short TTL. Cache is busted when a plan is assigned, changed, or completed. The check reads from cache first and only falls back to the database on a miss. This keeps the check sub-millisecond in the common case.

---

## 21. Business & Growth Considerations

### 21.1 Pricing Strategy

| Plan | Monthly | Yearly | Athletes | Target |
|---|---|---|---|---|
| **Athlete Pro** | €12 | €99 | 1 | Independent competitor wanting full analytics |
| **Club Starter** | €49 | €399 | 25 | Small clubs, school teams, community clubs |
| **Club Pro** | €129 | €999 | 100 | Established athletics clubs |
| **Enterprise** | Custom | Custom | Unlimited | National federations, professional organizations |

Annual billing saves ~2 months vs monthly. Offered at checkout with a clear comparison.

### 21.2 Growth Strategy

**Bottom-up:** Individual athletes sign up on Athlete Pro → they link their coach → coach sees the value → coach brings their entire squad → squad pressure drives the rest of the club onto the platform.

**Top-down:** Club director buys Club plan → invites 40 athletes and 4 coaches in one batch → entire club operational within a day.

**National team amplifier:** When a national federation adopts AthletIQ, all club athletes who make national squads are already on the platform (or join to be visible to national team coaches). This drives club adoption in federated markets.

**Network effects:** Squad comparison analytics and attendance tracking only deliver value when the majority of a squad is on the platform — this creates internal pressure within clubs for full adoption.

### 21.3 Retention Levers

- **Data depth:** Years of PRs, competition history, and training logs are highly sticky. Athletes will not abandon this data easily.
- **Weekly email digest:** Re-engages athletes who haven't logged in several days.
- **Recovery check-in streaks:** The daily wellness check-in creates a habit loop. Breaking a streak is psychologically costly.
- **Coach workflow dependency:** Once a coach uses the review queue and plan builder daily, switching means abandoning an established workflow and losing historical athlete data.
- **Competition PRs:** The officialized PR system (competition-only) gives the platform a uniquely trustworthy status compared to basic workout loggers.

### 21.4 Expansion Revenue

- Club grows past 25 athletes → auto-prompted to upgrade from Starter to Pro
- Annual billing discount drives annual conversions at renewal
- Add-on: AI summaries for Club Starter users who want the feature without upgrading the full plan
- Add-on: API access for technical users on Club Pro
- Enterprise upsell: clubs that hit 100 athletes or need white-labeling

### 21.5 Competitive Positioning

| Platform | Weakness | AthletIQ Advantage |
|---|---|---|
| TrainingPeaks | Generic — not track-specific. Expensive. | Purpose-built for track & field. Sprint splits, field event PRs, competition management. |
| Final Surge | Outdated UX. No real-time features. | Modern, fast, real-time WebSocket updates. |
| Strava | Social/fitness focus. No coach tools. | Coach-athlete workflow, load management, plans. |
| Spreadsheets | No automation, no alerts, no history analysis. | Automated ACWR, fatigue alerts, trend analytics. |

AthletIQ's moat is the combination of: track & field event specificity, the ACWR load engine, the coach-athlete workflow, and the data depth that accumulates over a season.

---

*Document Version: 3.0 — All product decisions confirmed and locked*
*Architecture Status: Implementation-ready*
*Next step: Begin V1 implementation starting with Identity domain and authentication*
ENDDOC
