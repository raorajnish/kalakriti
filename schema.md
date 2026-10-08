# KalaKriti — Database Schema (FINAL v1, 2026-10-08)

> Status: locked for the 15-day build. Changes go through the decision log.

## Conventions
- PKs are UUIDs (safe in URLs, no enumeration)
- `created_at` / `updated_at` on every table
- Money in paise (integers, never floats)
- Single `users` table — no separate profile table (artist profile = `bio` + `avatar_url`, too thin to justify a JOIN)

## users
| field | type | notes |
|---|---|---|
| id PK | uuid | |
| email UQ | varchar | login identity |
| password_hash | varchar | bcrypt/argon2, never plaintext |
| name | varchar | |
| phone | varchar, nullable, NOT unique | shared family numbers exist; unique would bite in testing |
| role | enum | VISITOR · ARTIST · ORGANIZER · ADMIN |
| city | varchar, nullable | drives "near me" discovery |
| avatar_url | varchar, nullable | |
| bio | text, nullable | artist profile |
| created_at, updated_at | timestamptz | |

## exhibitions
| field | type | notes |
|---|---|---|
| id PK | uuid | |
| organizer_id FK → users | uuid | who owns this show |
| title / theme / description | varchar / varchar / text | |
| venue_name / address / city | varchar | |
| latitude / longitude | float, nullable | map pin; radius search later |
| starts_at / ends_at | timestamptz | |
| daily_open / daily_close | time | e.g. 11:00–19:00 |
| entry_type | enum | FREE · TICKETED |
| ticket_price | int, nullable | paise; null when FREE |
| capacity | int, nullable | null = unlimited |
| banner_url | varchar, nullable | |
| status | enum | DRAFT · PUBLISHED · ONGOING · ENDED · CANCELLED |
| created_at, updated_at | timestamptz | |

## artworks
| field | type | notes |
|---|---|---|
| id PK | uuid | |
| artist_id FK → users | uuid | must have role ARTIST |
| title / description | varchar / text nullable | |
| image_url / thumbnail_url | varchar | original + generated thumb |
| medium | varchar | "Oil on canvas" |
| width_cm / height_cm | int, nullable | |
| year | int, nullable | |
| created_at | timestamptz | |

## open_calls
| field | type | notes |
|---|---|---|
| id PK | uuid | |
| exhibition_id FK → exhibitions | uuid | the job posting |
| brief | text | theme, what the curator wants |
| deadline | timestamptz | no applications after this |
| max_works_per_artist | int, default 3 | enforced at app level |
| entry_fee | int, default 0 | paise; stubbed in MVP |
| status | enum | OPEN · CLOSED |
| created_at | timestamptz | |

## applications
| field | type | notes |
|---|---|---|
| id PK | uuid | |
| open_call_id FK → open_calls | uuid | |
| artist_id FK → users | uuid | the job application |
| statement | text, nullable | artist's pitch |
| status | enum | SUBMITTED → UNDER_REVIEW → ACCEPTED / REJECTED / WAITLISTED → CONFIRMED |
| reviewer_note / reviewed_at | text, timestamptz, nullable | |
| created_at, updated_at | timestamptz | |
| UQ(open_call_id, artist_id) | | one application per artist per call |

## application_artworks (join)
| field | type | notes |
|---|---|---|
| application_id FK → applications | uuid | |
| artwork_id FK → artworks | uuid | |
| PK(application_id, artwork_id) | | |

## exhibition_artworks (join — the final hang list)
| field | type | notes |
|---|---|---|
| exhibition_id FK → exhibitions | uuid | |
| artwork_id FK → artworks | uuid | |
| display_order | int, default 0 | hang order |
| PK(exhibition_id, artwork_id) | | |

## rsvps (RSVPs + tickets, one table)
| field | type | notes |
|---|---|---|
| id PK | uuid | |
| exhibition_id FK → exhibitions | uuid | |
| visitor_id FK → users | uuid | |
| type | enum | RSVP · TICKET |
| headcount | int, default 1 | "I'm bringing 3 people" |
| qr_code UQ | varchar | scanned at the door |
| status | enum | CONFIRMED · CHECKED_IN · CANCELLED |
| created_at | timestamptz | |
| UQ(exhibition_id, visitor_id) | | one booking per visitor per show |

## notifications
| field | type | notes |
|---|---|---|
| id PK | uuid | |
| user_id FK → users | uuid | |
| type | varchar | APPLICATION_DECISION · EXHIBITION_REMINDER · … |
| title / body / link | varchar / text nullable | link deep-links into the app |
| is_read | boolean, default false | |
| created_at | timestamptz | |

## Relationships
- users 1—N exhibitions (organizer)
- users 1—N artworks (artist)
- users 1—N applications (artist)
- users 1—N rsvps (visitor)
- users 1—N notifications
- exhibitions 1—N open_calls
- exhibitions N—M artworks via exhibition_artworks
- open_calls 1—N applications
- applications N—M artworks via application_artworks
- exhibitions 1—N rsvps

## ER diagram
See `../kalakriti-design-doc.html` for the rendered diagram.
