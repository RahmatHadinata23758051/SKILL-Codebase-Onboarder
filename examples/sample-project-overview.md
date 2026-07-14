# Project Overview: BookStore API

*   **Analysis Date:** 2026-07-14 09:30
*   **Primary Language:** TypeScript
*   **Architecture Pattern:** Layered MVC
*   **Framework/Stack:** Next.js (App Router)
*   **Entry Points:**
    *   `src/app/api/route.ts`: Entry router endpoint configuration
*   **Database:** PostgreSQL (via Prisma ORM)

## Core Folder Structure

```text
bookstore-api/
├── src/
│   ├── app/                 # Presentation Layer (API Routing)
│   │   └── api/
│   ├── controllers/         # Presentation Layer (Request handling)
│   ├── services/            # Logic Layer (Business rules)
│   └── models/              # Data Layer (Database models)
└── prisma/                  # Data Layer (Prisma config & migrations)
```

## Architectural Summary

The project is structured as a standard three-tier layered MVC application built on Next.js App Router. The API requests are received at the route level, forwarded to controllers for validation, processed by service files that handle business logic, and mapped to the PostgreSQL database via Prisma models.

## Main Dependency Summary

*   `next`: ^14.2.0 (Framework)
*   `@prisma/client`: ^5.12.0 (ORM client)
*   `zod`: ^3.22.0 (Input validation library)
*   `typescript`: ^5.4.0 (Language compiler)
