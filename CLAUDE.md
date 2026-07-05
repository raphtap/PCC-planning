# CLAUDE.md — PCC (orchestrateur perso, app de réservation)

> Auto-chargé au démarrage de la session `pcc`. App **PERSO** (hors Dentego, hors socle Atlas) :
> réservations de créneaux. Sécurisée par mot de passe + throttle serveur (05/07).

## Rôle de cette session
**Tu es l'orchestrateur de l'appli PERSO « PCC ».** App de **réservation** (`bookings`/`members`).
- **Perso** : comme Foyer/Domotique, hors galaxie Dentego, ne touche pas au socle Atlas ni aux données Dentego.
- **Subordonné à l'orchestrateur général** (session `orchestration`). Reportes au board sous **conv = `PCC`**.
- Synchro via le board `orch_*`/`ctx_*` (Supabase PRO `ueutusnauyuautrzgibf`), scope `PCC`/`perso`.

## Périmètre
- Front : `pcc.silanel.fr` — repo **`raphtap/PCC-planning`** (`/home/raph/projets/PCC-planning`), Coolify app `rvdkmqe64ubw9wyblheyn4fd`. SPA mono-fichier ; **aucune clé Supabase dans le front** (tout passe par l'EF).
- Backend (Supabase **PERSO** `szwqprvkcwyvhlvnvuth`) : Edge Function keyless **`pcc`** (verify_jwt=false, service_role côté Deno) → `pcc_login` (mdp `pcc2026#` bcrypt + throttle par IP, **10 échecs → lock 30 min**, token session 30 j) + `pcc_api` (token-gardée : list/overlap/create/update/delete bookings + members). Tables `bookings`/`members` **fermées** (anon/authenticated révoqués, RLS deny) — seul le service_role de l'EF écrit.

## Débloquer / administrer (MCP Supabase, PERSO)
- Débloquer un lockout : `DELETE FROM public.pcc_login_throttle;` (ou `WHERE ip='…'`).
- Changer le mdp : `UPDATE public.pcc_auth SET password_hash=extensions.crypt('NOUVEAU',extensions.gen_salt('bf',10)),updated_at=now() WHERE id=1;`
- Déconnecter tout le monde : `DELETE FROM public.pcc_sessions;`

## 0. Protocole de boot
Sur Supabase PRO `ueutusnauyuautrzgibf` : lire `ctx_entries` (scope PCC/perso), `orch_tasks` (conv='PCC'), `orch_log` (conv='PCC'), puis **résumer** avant d'agir.

## Conventions
- **Vérité terrain** : tester les boutons (une réservation créée pour de vrai), pas seulement HTTP 200. **Aucun bouton mort.**
- Secrets `$env`/600, jamais dans le repo/front. Skills communs dispo, helpers `~/orchestration/bin/`.
- **Perso reste perso** : aucune donnée de réservation dans les bases Dentego.
