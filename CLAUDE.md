# CLAUDE.md — PCC (orchestrateur perso, app de réservation)

> Auto-chargé au démarrage de la session `pcc`. App **PERSO** (hors Dentego, hors socle Atlas) :
> réservations de créneaux. Sécurisée par mot de passe + throttle serveur (05/07).

## Rôle de cette session
**Tu es l'orchestrateur de l'appli PERSO « PCC ».** App de **réservation** (`bookings`/`members`).
- **Perso** : comme Foyer/Domotique, hors galaxie Dentego, ne touche pas au socle Atlas ni aux données Dentego.
- **Subordonné à l'orchestrateur général** (session `orchestration`). Reportes au board sous **conv = `PCC`**.
- Synchro via le board `orch_*`/`ctx_*` (Supabase PRO `ueutusnauyuautrzgibf`), scope `PCC`/`perso`.

## Périmètre
- Front : `pcc.silanel.fr` — repo **`raphtap/PCC-planning`** (`/home/raph/projets/PCC-planning`). SPA mono-fichier ; **aucune clé Supabase dans le front** (tout passe par l'EF). Fix déployés le 19/07 : échappement des noms de membres (anti bouton-mort/injection) + contrôle de capacité 6/6 sur les réservations récurrentes (fin du surbooking silencieux).
- Backend (Supabase **PERSO** `szwqprvkcwyvhlvnvuth`) : Edge Function keyless **`pcc`** (verify_jwt=false, service_role côté Deno) → `pcc_login` (mot de passe **bcrypt** — le mdp courant n'est **PAS** dans le repo, voir mémoire de session / board ; l'ancien `pcc2026#` inscrit ici était **périmé** (rotation 06/07) et a été retiré du doc le 19/07 — throttle par IP, **10 échecs → lock 30 min**, token session 30 j) + `pcc_api` (token-gardée : list/overlap/create/update/delete bookings + members). Tables `bookings`/`members` **+** `pcc_auth`/`pcc_sessions`/`pcc_login_throttle` **fermées** (anon/authenticated révoqués — durcissement 17/07, RLS deny) — seul le service_role de l'EF écrit.

## Hébergement (migration PERSO du 23/07)
- **Serveur PERSO OVH `151.80.87.117`** (`ssh vps-perso`, Coolify perso `http://localhost:8000`, token `/root/.coolify_perso_token`, projet **PERSO** `chuusbnry1k7cv7v5fz3hwh5`) — app PCC **`fdks2rdlxthu8p9iqh61otyr`**, source = repo **public** GitHub, build **nixpacks static** (nginx:alpine, port 80). Déployée + E2E validé en Host forcé le 23/07.
- ⚠️ **Pas d'auto-deploy** sur le perso (pas de GitHub App → pas de webhook) : après un `git push`, déclencher `GET /api/v1/deploy?uuid=fdks2rdlxthu8p9iqh61otyr` sur le Coolify perso.
- Ancien hébergement **PRO** (VPS3 `141.95.170.171`, app `rvdkmqe64ubw9wyblheyn4fd`, auto-deploy actif) : à retirer par le général après bascule DNS. Le DNS est **géré par le général**, pas par cette session.

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
