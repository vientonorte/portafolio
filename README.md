# Portafolio — Plan de implementación

> Consolidar el portafolio profesional en un único repositorio desplegado en GitHub Pages.

## Situación actual

| Carpeta | Contenido | Git | Estado |
|---------|-----------|-----|--------|
| `Portafolio/` | Carpeta organizadora con Dashboard (HTML vanilla) y subcarpeta mi-portafolio | `.git` corrupto (sin refs/) | ❌ Roto |
| `Portafolio/mi-portafolio/` | Clon de `vientonorte/mi-portafolio` (HTML + Bootstrap) | ✅ remote: vientonorte/mi-portafolio | Obsoleto |
| `Portafolio/Dashboard/` | Dashboard "Mis números" (HTML + Material Icons) | Sin repo propio | Prototipo |
| `mi-portafolio/` (raíz) | Otro clon de `vientonorte/mi-portafolio` | ✅ remote: vientonorte/mi-portafolio | Obsoleto |
| `Portafolio Lead UX/` | **Código Figma exportado** — React 19 + Radix/shadcn + Vite | Sin git (solo local) | ✅ Build existente (3.6 MB) |

### Proyecto Figma source

- **URL:** `https://glade-relink-16453631.figma.site` (clave: `indescifrable`)
- **Exportado a:** `Portafolio Lead UX/` vía Figma Code Bundle
- **Stack:** React 19 + TypeScript + Radix UI + shadcn/ui + Tailwind CSS v4 + Vite 6
- **Secciones:** Hero, About, ImpactStats, ProjectsHub (SURA, Transvip, Karri), Experience, Contact, Design System, Case Studies
- **Features:** i18n (ES/EN), a11y (skip-to-content, focus-visible), responsive, rutas SPA

---

## Plan de implementación

### Fase 1 — Preparar repositorio (Must · Effort S)

```bash
# 1. Limpiar carpeta Portafolio/ (git corrupto)
cd /Users/ro/Documents/GitHub/Portafolio
rm -rf .git .vscode .DS_Store

# 2. Eliminar subcarpetas obsoletas
rm -rf mi-portafolio Dashboard

# 3. Mover el código Figma al repo
cp -R "/Users/ro/Documents/GitHub/Portafolio Lead UX/src" .
cp -R "/Users/ro/Documents/GitHub/Portafolio Lead UX/build" .
cp "/Users/ro/Documents/GitHub/Portafolio Lead UX/package.json" .
cp "/Users/ro/Documents/GitHub/Portafolio Lead UX/package-lock.json" .
cp "/Users/ro/Documents/GitHub/Portafolio Lead UX/vite.config.ts" .
cp "/Users/ro/Documents/GitHub/Portafolio Lead UX/index.html" .

# 4. Inicializar como repo limpio apuntando al remote existente
git init
git remote add origin https://github.com/vientonorte/mi-portafolio.git
```

> **Alternativa:** Crear nuevo repo `vientonorte/portafolio` y archivar `mi-portafolio`.

### Fase 2 — Build local y validación (Must · Effort S)

```bash
# Instalar dependencias
npm install

# Build de producción
npx vite build --base=/mi-portafolio/

# Verificar build
npx vite preview
```

**Validaciones:**
- [ ] Todas las secciones renderizan (Hero → Contact)
- [ ] Navegación SPA funciona (Design System, Case Studies, Process Detail)
- [ ] i18n ES/EN alterna correctamente
- [ ] Responsive: mobile < 768px, tablet, desktop
- [ ] Assets (imágenes Figma) cargaron sin 404
- [ ] a11y: skip-to-content, landmarks, focus-visible

### Fase 3 — Deploy GitHub Pages (Must · Effort S)

**Opción A — GitHub Actions (recomendada):**

```yaml
# .github/workflows/deploy.yml
name: Deploy to Pages
on:
  push:
    branches: [main]
permissions:
  contents: read
  pages: write
  id-token: write
jobs:
  build-and-deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 20
          cache: npm
      - run: npm ci
      - run: npx vite build --base=/mi-portafolio/
      - uses: actions/upload-pages-artifact@v3
        with:
          path: dist
      - uses: actions/deploy-pages@v4
```

**Opción B — Deploy estático (build/ a gh-pages):**

```bash
# Push del build existente a rama gh-pages
git checkout -b gh-pages
cp -R build/* .
git add -A
git commit -m "deploy: portafolio Lead UX from Figma export"
git push origin gh-pages
```

### Fase 4 — Limpieza de duplicados (Should · Effort S)

| Acción | Carpeta/Repo | Resultado |
|--------|-------------|-----------|
| Eliminar carpeta local | `Portafolio Lead UX/` | Se movió al repo consolidado |
| Eliminar carpeta local | `Portafolio/` (original corrupto) | Reemplazado por repo limpio |
| Archivar o redirigir | `mi-portafolio/` (local) | Apunta al mismo remote |
| Actualizar hub | `vientonorte.github.io` | Cambiar link del portafolio |

### Fase 5 — Documentación (Should · Effort S)

- [ ] README.md con: propósito, stack, cómo correr local, cómo deployar
- [ ] Actualizar `vientonorte.github.io/index.html` con URL correcta del portafolio
- [ ] Agregar `_redirects` o meta-refresh si cambia la URL del portafolio

---

## Decisiones pendientes

| Decisión | Opciones | Impacto |
|----------|----------|---------|
| ¿Repo destino? | Reusar `vientonorte/mi-portafolio` / Crear nuevo `vientonorte/portafolio` | URL pública cambia |
| ¿Base path? | `/mi-portafolio/` (reusar URL) / `/portafolio/` (nuevo) / `/` (si es .github.io) | Vite `base` config |
| ¿CI/CD o build manual? | GitHub Actions (auto) / Push estático (manual) | Mantenibilidad |
| ¿Preservar `node_modules` en repo? | NO — solo `package.json` + `package-lock.json` | CI instala deps |
| ¿Eliminar `Portafolio Lead UX/`?| Sí después de migrar / No como backup | Limpieza workspace |

---

## Diagrama de estado final

```
vientonorte/mi-portafolio (o /portafolio)
├── .github/workflows/deploy.yml
├── index.html
├── package.json
├── vite.config.ts
├── src/
│   ├── App.tsx              (Hero, About, ImpactStats, Projects, Experience, Contact)
│   ├── components/          (Radix + shadcn design system)
│   ├── pages/               (DesignSystem, CaseStudies, ProcessDetail)
│   ├── data/                (projects-data, karri-projects)
│   └── styles/
└── README.md

Deploy: push to main → GitHub Actions → vite build → GitHub Pages
URL:    https://vientonorte.github.io/mi-portafolio/
```
