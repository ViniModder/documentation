# 🚀 Deploy Rápido - GitHub Pages

Siga estes passos para hospedar a documentação gratuitamente:

## 1️⃣ Instalar Dependências

```bash
pip install mkdocs mkdocs-material
```

## 2️⃣ Testar Localmente

```bash
mkdocs serve
```

Acesse: http://127.0.0.1:8000

## 3️⃣ Configurar mkdocs.yml

Substitua `SEU_USUARIO` pelo seu nome do GitHub:

```yaml
site_url: https://SEU_USUARIO.github.io/combo-lategame
repo_url: https://github.com/SEU_USUARIO/combo-lategame
```

## 4️⃣ Criar Repositório no GitHub

1. Acesse https://github.com/new
2. Nome: `combo-lategame`
3. Crie o repositório

## 5️⃣ Fazer Push

```bash
git init
git add .
git commit -m "Initial commit"
git remote add origin https://github.com/SEU_USUARIO/combo-lategame.git
git branch -M main
git push -u origin main
```

## 6️⃣ Ativar GitHub Pages

1. Vá em **Settings** → **Pages**
2. **Source**: Selecione **GitHub Actions**
3. Clique em **Save**

## 7️⃣ Acompanhar Deploy

Vá em **Actions** e aguarde o workflow terminar (✅ verde)

## 8️⃣ Acessar

```
https://SEU_USUARIO.github.io/combo-lategame
```

---

📖 **Guia completo**: [DEPLOY.md](DEPLOY.md)
