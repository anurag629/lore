# Lore (Wrapper Repository)

[![Hackathon](https://img.shields.io/badge/Encode%20Club-Surreal%20World%20Assets%20Buildathon-9A3CFF)](https://www.encodeclub.com/programmes/surreal-world-assets-buildathon-2)
[![Frontend](https://img.shields.io/badge/frontend-lore__frontend-0A7EA4?logo=github&logoColor=white)](https://github.com/anurag629/lore_frontend)
[![Backend](https://img.shields.io/badge/backend-lore__backend-3E7D28?logo=github&logoColor=white)](https://github.com/anurag629/lore_backend)
[![Submodules](https://img.shields.io/badge/git-submodules-555?logo=git&logoColor=white)](https://git-scm.com/book/en/v2/Git-Tools-Submodules)
[![Clone Recursive](https://img.shields.io/badge/clone---recursive-2ea44f?logo=git&logoColor=white)](#clone-with-submodules)

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

