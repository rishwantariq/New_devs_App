# property revenue dashboard – what got fixed

## 1. cross-tenant data leakage (ocean rentals)

ocean was sometimes seeing sunset's revenue numbers after a refresh. the cache key only used property_id, so both tenants were hitting the same cache entry (they both have prop-001 etc). fixed by including tenant in the key: `revenue:v2:{tenant_id}:{property_id}` in cache.py.

## 2. wrong march totals (sunset)

march numbers didn't match what sunset had. the revenue query had no month filter – it was always returning all-time totals even though the UI says "monthly". also no timezone handling for properties. added month/year params to the dashboard summary api and made the query filter by date. frontend now passes month: 3, year: 2024 for march.

## 3. totals off by cents

finance was seeing tiny rounding differences. we were doing float() on the total directly. now we round with Decimal to 2 decimals (ROUND_HALF_UP) before converting to float in the dashboard api.

## 4. cors blocking login

login was failing when using the app from a local network url like 192.168.x.x:5173. added allow_origin_regex in main.py so localhost and lan ips work.

## 5. fallback mock data not tenant-scoped

when the db was down, the mock data was keyed by property_id only. so tenants could see each other's fallback numbers. changed reservations.py so the mock structure is mock_data[tenant_id][property_id].

## 6. database pool not connecting

the pool was using config that didn't exist so it never connected and we always got mock data. fixed database_pool.py to use settings.database_url and convert it to the async postgres format (postgresql+asyncpg).

## 7. dashboard showing wrong properties

the property dropdown used a hardcoded list, so client a could pick client b's properties. added a /dashboard/properties endpoint that returns only the current tenant's properties. dashboard fetches from that now.

## 8. secureapi breaking with localauth

secureApi.ts had type errors and the supabase.from intercept was blowing up because LocalAuthClient doesn't have a from method. added a guard for that, fixed the session types so it works with both localauth and supabase, and cleaned up the unsubscribe logic.
