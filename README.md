# The Local Loop: System Architecture Case Study

**Project Status:** Production Architecture Specifications (Public Spec) | Live application is in active development (Private Repo).

### Premium Multi-Tenant Restaurant Management & Discovery Platform

The Local Loop is designed to give restaurant owners in smaller towns a dedicated space to advertise their businesses, build patron awareness, and promote events and daily specials. 

The concept was born from a consumer-first problem: the need for a unified platform where users can effortlessly discover which local establishments are hosting specials on any given day. As the project evolved, the focus expanded into engineering a comprehensive portal where business owners can scale their digital presence, manage multi-unit operational data, and ultimately track critical statistics like user engagement, patron attendance, and advertisement success.

To support this real-world operational scale, the platform is built as a high-performance, multi-tenant SaaS architecture. With a dual emphasis on robust database integrity and a context-aware user experience, the application bridges complex backend relational mapping with razor-sharp frontend control.

---

> ### 🔒 Code Privacy & Intellectual Property Notice
> The core codebase, database schemas, and proprietary business relations for this restaurant optimization engine sit securely inside a private repository. This public repository serves as a system design portfolio piece detailing the multi-tenant security architecture, asset processing pipelines, and relational database paradigms used to construct the platform.

---

## 🏗️ Technical Architecture & Stack

* **Frontend Engine:** React, TypeScript, Tailwind CSS
* **Backend & Database (BaaS):** Supabase (PostgreSQL relational engine, Row-Level Security isolation, automated multi-tenant triggers, and custom security gateway functions)
* **Media Processing:** Client-side hardware-accelerated Canvas scaling pipeline
* **AI-Assisted Prototyping and Architecture:**
  * Accelerated MVP development using Lovable for rapid full-stack prototyping.
  * Leveraged Gemini to migrate and translate legacy TSQL patterns into optimized PostgreSQL.
  * Utilized AI as a planning and analysis tool to critique architectural roadmaps.

---

## 🔄 The Content & Security Lifecycle

The operational architecture splits data transitions into isolated public distribution, encrypted tenant routing, and hardware-level asset optimization layers:

```text
[ 0. PUBLIC DISCOVERY LAYER ]
Anonymous internet traffic (anon) hits the core public feed.
Database permits read-only (SELECT) access to specials, tags, and directories.
                               |
                               v
[ 1. AUTHENTICATED WORKSPACE ENTRY ]
Restaurant Owner logs in -> Acquires cryptographically signed JWT token.
PostgreSQL intercepts token claims via session memory to assert identity.
                               |
                               v
[ 2. CLIENT-SIDE EDGE MEDIA INTERCEPT ]
Owner uploads a new daily special image (e.g., a raw 10MB device photo).
Application prevents direct storage transmission and mounts a hidden Canvas.
  - Forces strict 16:9 crop boundaries.
  - Micro-scales physical geometry down to a maximum 1200px width threshold.
  - Compresses matrix representation to 75% quality JPEG.
                               |
                               v
[ 3. ATOMIC STORAGE SYNCHRONIZATION ]
Frontend triggers an atomic pipeline execution:
  1. Purges historical media asset linked to target row from Supabase Bucket.
  2. Ingests new high-performance, sub-100KB compressed asset.
                               |
                               v
[ 4. RECURSION-FREE DATABASE ASSIGNMENT ]
Database executes mutation (INSERT/UPDATE).
RLS passes current user UUID through a high-speed SECURITY DEFINER function:
  - Validates user role status.
  - Confirms ownerid matches table destination.
  - Commits row to database or executes global admin override shortcut.
```

---

## 🗄️ Database Architecture & Multi-Tenant Blueprint

Data relationships are isolated using a five-tiered relational matrix designed around explicit permission silos:

### 1. The Core Tenant Infrastructure (`owners` & `restaurants`)
* **`owners` Table:** Maps individual authenticated operators directly to system roles (`owner`, `admin`). The primary key directly mirrors the cryptographic user ID generated within Supabase’s internal authentication schema.
* **`restaurants` Table:** Serves as the primary parent entity for physical locations. It is linked back to the manager via a strict foreign key constraint (`ownerid`). This mapping guarantees that multi-branch networks map cleanly to a single owner profile.

### 2. The Operational Content Layer (`specials` & `master_tags`)
* **`specials` Table:** Tracks live promotional events, price mechanics, and asset references. This table does not explicitly track `ownerid`. Instead, rows map back dynamically to a parent `restaurant_id`, enabling deep multi-tenant security verification via relational join lookups.
* **`master_tags` Table:** A centralized lookup dictionary containing static categorization labels (e.g., "Veggie", "Happy Hour", "Steakhouse").

### 3. The Decoupled Consumer Directory (`customers`)
* **`customers` Table:** Houses end-user tracking variables (UUIDs and public identification handles). This directory is completely isolated from administrative business parameters to protect consumer identity matrices.

---

## ⚡ Engineering Highlights

### 🖼️ 1. Edge-Optimized Canvas Media Compression
To guarantee predictable storage consumption metrics and blazing-fast feed rendering speeds on mobile networks, the application implements a frontend-driven image interception pipeline.

When an operator uploads a raw smartphone camera asset, the application blocks the network thread and transfers the raw data stream into an in-memory canvas context. The pipeline forces a locked 16:9 aspect ratio boundary, scales the physical dimensions linearly until the horizontal resolution hits a maximum 1200px constraint, and converts the output to a 75% quality JPEG.

This approach minimizes data transmission overhead, scales storage costs linearly, and guarantees that every single image asset resides in the storage bucket as a featherweight, high-performance web asset (typically scaling from 10MB down to sub-100KB layers).

### 🔐 2. Recursion-Free Administrative Override (Security Architecture)
Standard multi-tenant database designs often cause a classic PostgreSQL failure mode: Row-Level Security infinite recursion loops. When an RLS policy on a user table recursively queries that exact same user table to verify if the querying entity has an 'admin' role, the execution stack crashes.

The Local Loop bypasses this vulnerability completely by isolating administrative status verification inside a dedicated database function:

```sql
CREATE OR REPLACE FUNCTION public.is_admin()
RETURNS boolean 
SECURITY DEFINER 
SET search_path = public 
AS $$
BEGIN
  RETURN EXISTS (
    SELECT 1 FROM public.owners 
    WHERE id = auth.uid() AND role = 'admin'
  );
END;
$$ LANGUAGE plpgsql;
```

Because this helper function is marked as `SECURITY DEFINER`, it executes with elevated system privileges, checking the root database layer directly and bypassing RLS evaluation paths.

The application then deploys standalone, high-priority RLS gates across all tables (`FOR ALL TO authenticated USING (public.is_admin())`). Because PostgreSQL evaluates multiple matching rules using an `OR` gate logic format, administrators gain instant global override privileges, allowing them to perform full system CRUD mutations while keeping standard tenant filters isolated.

### 🛡️ 3. Decoupled Multi-Tenant Isolation
For restaurant operations, security must be completely invisible yet absolutely unbreachable. The platform ensures strict multi-tenancy at the database level. For example, the `specials` table implements an RLS rule that executes relational join logic across tables before confirming any mutations:

```sql
CREATE POLICY "Owners can manage specials for restaurants they own" 
ON public.specials FOR ALL TO authenticated 
USING (
    EXISTS (
        SELECT 1 FROM public.restaurants 
        WHERE restaurants.id = specials.restaurant_id 
        AND restaurants.ownerid = auth.uid()
    )
);
```

This logic ensures that even if a malicious user manipulates frontend component states or intercepts API variables via network tools, the underlying database will systematically reject any operation that attempts to view, modify, or erase data belonging to a competing restaurant tenant.

---

## 🚀 Future Enhancements (Post-MVP Roadmap)

* **Administrative Change Request Pipeline:** Implements an asynchronous validation sequence that catches owner mutations and routes them to a moderation table for admin verification prior to public deployment.
* **Subscription Tier Management:** Integrating billing gateways directly at the database level to restrict total branch configurations and maximum promotional counts dynamically based on active payment tiers.
