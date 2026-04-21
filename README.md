# Case_study_Splunk_2# 🔍 Splunk – AWS Log Analysis

## 📌 Úvod

Tento dokument popisuje základní práci ve Splunku při analýze AWS logů.

Cílem je pochopit:
- jak se vyhledává v logách
- jak se analyzuje provoz
- jak poznat podezřelou aktivitu
- jak přemýšlí SOC analytik

---

## 🧭 Splunk – základní vyhledávání

### 🔎 New Search
Ve Splunku pracujeme s dotazy nad indexy:

```spl
index=aws
📊 Možnosti vizualizace
timeline zobrazení
logaritmická škála
lineární škála
filtrování podle času
🏢 Kontext

V reálné firmě:

logy jsou klíčové pro bezpečnost
obsahují informace o přístupech, uživatelích a API voláních
správná analýza = základ SOC (Security Operations Center)
🧪 EXERCISE 1 – AWS log analýza
1️⃣ Filtrování interního provozu
index=aws sourceIPAddress!=109.81.173.230 | table sourceIPAddress

👉 výstup: seznam IP adres

2️⃣ Nalezení IP s vysokou aktivitou
index=aws sourceIPAddress=88.208.91.78 
| stats count by eventName 
| sort -count

👉 nalezeno ~30 000 logů

📊 Typy operací

Typické eventy:

Describe*
List*
Get*
📌 Význam

✔ read-only operace
✔ žádné změny infrastruktury
✔ žádné vytváření uživatelů nebo klíčů

📍 Další IP analýza
index=aws sourceIPAddress=212.20.115.63 
| stats count by eventName 
| sort -count
📌 Výstup
GetRole
ListAccessKeys
GetLoginProfile
ListMFADevices
ListGroupsForUser
ListSigningCertificates
🚨 Interpretace

Tato aktivita odpovídá:

👉 Identity reconnaissance

Zjišťování:

access keys
MFA zařízení
IAM role
uživatelské účty
oprávnění
🔥 Možné scénáře
🟢 Legit scénář
security audit (Prowler, ScoutSuite)
compliance kontrola
interní script
🟡 Interní uživatel
debugging
ruční kontrola prostředí
🔴 Útočník
reconnaissance po získání přístupu
mapování AWS prostředí
⏱️ Analýza chování v čase
index=aws sourceIPAddress=212.20.115.63
| bin _time span=1m
| stats count by _time
📌 Interpretace
Chování	Význam
rovnoměrné	script / tool
spike	podezřelá aktivita
nepravidelné	scanning
👤 Analýza identity
index=aws sourceIPAddress=212.20.115.63
| stats count by userIdentity.type, userIdentity.arn
📌 Výsledek
AssumedRole → standardní AWS best practice
IAMUser → přímý uživatel
🔐 Co znamená AssumedRole

Např.:

assumed-role/administrator/user

👉 znamená:

uživatel se přihlásil
převzal IAM roli
pracuje přes AWS API
🧠 Finální verdikt

👉 Aktivita je:

🟢 legitimní

Důvody:
pouze read-only operace
žádné Create / Delete / Put operace
používány IAM role
konzistentní chování
💡 SOC mindset

❌ Junior přístup:

„hodně logů = incident“

✅ Analytický přístup:

„jaký je kontext, kdo to je a co dělá?“

🚨 Red flags

Sledovat:

Create*
Delete*
Put*
neznámé IP adresy
neznámé IAM identity
náhlé traffic spikes
🧪 Shrnutí

✔ AWS log analýza
✔ Splunk querying
✔ detekce reconnaissance
✔ IAM role understanding
✔ SOC mindset

🧠 Klíčová myšlenka

👉 Velké množství logů ≠ incident

Správná analýza vždy odpovídá:

kdo to je
co dělá
odkud to dělá
jak často
s jakými právy
