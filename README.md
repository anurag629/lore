# Lore (Wrapper Repository)

This is the wrapper repo for the Surreal World Assets Buildathon project. It tracks the frontend and backend as submodules so judges and collaborators can pull everything from a single place.

## Repos
- Frontend: `lore_frontend` → https://github.com/anurag629/lore_frontend
- Backend: `lore_backend` → https://github.com/anurag629/lore_backend
- Hackathon: https://www.encodeclub.com/programmes/surreal-world-assets-buildathon-2

## Clone (with submodules)
```bash
git clone --recursive https://github.com/anurag629/lore.git
cd lore
```

If you already cloned without `--recursive`, initialize submodules with:
```bash
git submodule update --init --recursive
```

## Structure
- `lore_frontend/` – Next.js 16 app (Groups, Disputes, Permissions, Royalties, multi-parent remixes, AI-assisted flows). See its README for setup.
- `lore_backend/` – Django REST API (assets, multi-parent derivatives, groups, disputes, permissions, royalty payments, AI services). See its README for setup.

## Updating submodules to latest
```bash
git submodule update --remote
git add .
git commit -m "Update submodules to latest"
git push
```

## Running locally
- Frontend: `cd lore_frontend && npm install && npm run dev`
- Backend: `cd lore_backend && python -m venv venv && venv\Scripts\activate && pip install -r requirements/base.txt && python manage.py migrate && python manage.py runserver`

Refer to each submodule’s README for environment variables and detailed steps.

