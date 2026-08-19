# =============================================================================
#  Frontend — Vue 3 (Vite) sur Node.js 20 (Alpine)
#  Sert l'interface en mode développement avec rechargement à chaud (HMR).
# =============================================================================
FROM node:22-alpine

WORKDIR /app

# En développement, le code est monté via docker-compose (bind mount) et les
# dépendances s'installent au premier lancement :  npm install
# (voir README.md, section « Premier démarrage »).

# Port par défaut du serveur de développement Vite
EXPOSE 5173

# --host : rend le serveur accessible depuis l'extérieur du conteneur
CMD ["sh", "-c", "npm install --legacy-peer-deps && npm run dev -- --host"]
