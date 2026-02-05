# CLAUDE.md - Security-Aware Development

Bij elke code-interactie in dit project pas je automatisch security-analyse toe. Je schrijft geen regel code zonder deze te toetsen aan onderstaande patronen. Je meldt kwetsbaarheden proactief, ook als de gebruiker er niet om vraagt.

---

## Gedrag

### Bij het LEZEN van code
Scan automatisch op:
- Hardcoded secrets (API keys, wachtwoorden, tokens)
- User input die ongefilterd doorstroomt
- Ontbrekende authenticatie/autorisatie
- Onveilige dependencies
- Misconfiguraties

### Bij het SCHRIJVEN van code
Pas automatisch toe:
- Input validatie op elke systeemgrens
- Output encoding bij elke render
- Parameterized queries bij elke database-interactie
- Least privilege bij elke permissie
- Fail secure bij elke error

### Bij het REVIEWEN van code
Controleer elk diff op:
- Nieuwe aanvalsvectoren
- Regressies in bestaande security
- Secrets in commits
- Dependency wijzigingen

---

## Vulnerability Patronen

### Injection

**SQL Injection** - Herken dit:
```
# KWETSBAAR
query = f"SELECT * FROM users WHERE id = {user_input}"
query = "SELECT * FROM users WHERE id = " + params[:id]
db.query("SELECT * FROM users WHERE name = '" + name + "'")

# VEILIG
cursor.execute("SELECT * FROM users WHERE id = %s", (user_input,))
User.where(id: params[:id])
db.query("SELECT * FROM users WHERE name = ?", [name])
```

**Command Injection** - Herken dit:
```
# KWETSBAAR
os.system(f"ping {user_input}")
exec("ls " + params[:dir])
child_process.exec(`convert ${filename}`)

# VEILIG
subprocess.run(["ping", user_input], shell=False)
system("ls", params[:dir])
execFile("convert", [filename])
```

**XSS** - Herken dit:
```
# KWETSBAAR
innerHTML = user_input
<div>{dangerouslySetInnerHTML}</div>
res.send("<p>" + comment + "</p>")
template: `<div>${userContent}</div>`

# VEILIG
textContent = user_input
<div>{escaped_output}</div>
res.send("<p>" + escapeHtml(comment) + "</p>")
```

**Path Traversal** - Herken dit:
```
# KWETSBAAR
file = open(f"/uploads/{filename}")
File.read(params[:path])
fs.readFile(req.query.file)

# VEILIG
path = os.path.join(UPLOAD_DIR, os.path.basename(filename))
safe_path = File.expand_path(path, ALLOWED_DIR)
if (!resolved.startsWith(ALLOWED_DIR)) throw new Error()
```

### Authenticatie & Sessies

**Herken deze fouten:**
```
# KWETSBAAR
if password == stored_password:              # Plaintext vergelijking
token = base64.encode(user_id)              # Voorspelbaar token
session.cookie.secure = false               # Cookie over HTTP
jwt.decode(token, algorithms=["none"])      # Algorithm none attack
```

**Verplichte patronen:**
```
# VEILIG
bcrypt.checkpw(password, hashed)            # Hash vergelijking
token = secrets.token_urlsafe(32)           # Cryptografisch random
session.cookie.secure = true                # HTTPS only
session.cookie.httpOnly = true              # Geen JS toegang
session.cookie.sameSite = "strict"          # CSRF bescherming
jwt.decode(token, key, algorithms=["RS256"])# Specifiek algoritme
```

### Autorisatie

**Herken deze fouten:**
```
# KWETSBAAR - IDOR
GET /api/users/123/profile                  # Geen eigenaar-check
DELETE /api/posts/{id}                      # Geen permissie-check

# KWETSBAAR - Privilege escalation
if user.role != "admin":                    # Client-side check
  hide_button()
```

**Verplichte patronen:**
```
# VEILIG
if resource.owner_id != current_user.id:
    raise Forbidden()

@require_permission("admin")
def delete_user(user_id):
    ...
```

### Cryptografie

**Herken deze fouten:**
```
# KWETSBAAR
md5(password)                               # Zwakke hash
DES.encrypt(data)                           # Verouderd algoritme
random.randint(0, 999999)                   # Niet-crypto random
key = "hardcoded-secret-key"                # Hardcoded key
```

**Verplichte patronen:**
```
# VEILIG
bcrypt.hash(password, rounds=12)            # Of argon2
AES-256-GCM.encrypt(data)                   # Moderne cipher
secrets.token_urlsafe(32)                   # Crypto random
key = os.environ["SECRET_KEY"]              # Env variable
```

### Data Exposure

**Herken deze fouten:**
```
# KWETSBAAR
logger.info(f"Login: {username}:{password}")  # Credentials in logs
return jsonify(user.__dict__)                 # Hele object terug
console.log("API key:", apiKey)               # Secrets in console
```

**Verplichte patronen:**
```
# VEILIG
logger.info(f"Login attempt: {username}")
return jsonify(user.to_public_dict())
# Nooit secrets loggen
```

---

## Security Headers

Bij elke HTTP response, controleer op:
```
Content-Security-Policy: default-src 'self'
X-Content-Type-Options: nosniff
X-Frame-Options: DENY
Strict-Transport-Security: max-age=31536000; includeSubDomains
Referrer-Policy: strict-origin-when-cross-origin
Permissions-Policy: camera=(), microphone=(), geolocation=()
```

---

## Dependency Check

Bij elke package/import wijziging:
- Is het package bekend en vertrouwd?
- Heeft het bekende CVEs?
- Is het actief onderhouden?
- Heeft het te brede permissies?

---

## Secrets Detectie

Blokkeer ALTIJD commits die bevatten:
```
# Patronen
AWS_ACCESS_KEY_ID=AKIA...
PRIVATE_KEY=-----BEGIN RSA PRIVATE KEY-----
password\s*=\s*["'][^"']+["']
api[_-]?key\s*=\s*["'][^"']+["']
token\s*=\s*["'][a-zA-Z0-9]{20,}["']
DATABASE_URL=postgres://user:pass@
```

Verwijs naar `.env.example` met placeholder waarden.

---

## Rapportage

Bij elke gevonden kwetsbaarheid, meld:
1. **Wat**: Welk type kwetsbaarheid
2. **Waar**: Exact bestand en regelnummer
3. **Risico**: Hoog/Middel/Laag
4. **Fix**: Concrete code-oplossing
5. **Waarom**: Wat een aanvaller ermee kan

Formaat:
```
SECURITY: [HOOG] SQL Injection in api/users.py:42
  → user_input wordt ongefilterd in query gebruikt
  → Aanvaller kan volledige database lezen/wijzigen
  → Fix: Gebruik parameterized query
```

---

*Deze CLAUDE.md transformeert elke AI code-interactie in een security audit. Drop in elk project, klaar.*
