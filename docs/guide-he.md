# מדריך: איך להעביר אפליקציה מ-Lovable / Base44 ל-Next.js עם סוכן AI

## למי המדריך הזה

בנית אפליקציה ב-Lovable או Base44 ועכשיו אתה צריך:
- SEO אמיתי (שגוגל יראה את התוכן שלך)
- Server-Side Rendering
- Metadata לכל עמוד
- Sitemap ו-robots.txt
- Auth שעובד בצד שרת
- Deploy מקצועי ב-Vercel

המדריך הזה מראה איך לעשות את כל זה עם סוכן AI ב-Claude Code שכתבתי, שעושה את כל העבודה בשבילך.

---

## מה צריך לפני שמתחילים

### 1. Node.js (גרסה 18 ומעלה)

בדוק אם מותקן:
```bash
node -v
```

אם לא, תוריד מפה: https://nodejs.org

### 2. חשבון Supabase

https://supabase.com — הרשמה חינם. אם הפרויקט שלך כבר רץ על Supabase (כמו ב-Lovable), אתה כבר מסודר. אם זה Base44, תצטרך ליצור פרויקט Supabase חדש.

### 3. חשבון Vercel

https://vercel.com — הרשמה חינם. שם הפרויקט יעלה לפרודקשן.

### 4. Claude Code

Claude Code זה CLI של Anthropic שנותן ל-Claude לעבוד ישירות בטרמינל שלך — לקרוא קבצים, לערוך קוד, להריץ פקודות. בעצם מתכנת AI שיושב לך בפרויקט.

**התקנה:**
```bash
npm install -g @anthropic-ai/claude-code
```

**מפתח API:**
1. נכנסים ל-https://console.anthropic.com
2. יוצרים חשבון (או נכנסים)
3. הולכים ל-API Keys ויוצרים מפתח חדש
4. מגדירים בטרמינל:
```bash
export ANTHROPIC_API_KEY=your-key-here
```

כדי שזה ישאר גם אחרי סגירת הטרמינל, תוסיף את השורה הזו לקובץ `~/.zshrc` (מק) או `~/.bashrc` (לינוקס):
```bash
echo 'export ANTHROPIC_API_KEY=your-key-here' >> ~/.zshrc
source ~/.zshrc
```

**בדיקה שזה עובד:**
```bash
mkdir test-project && cd test-project
claude
```

אם נפתח Claude Code ואתה רואה prompt, הכל עובד. כתוב `exit` לצאת.

---

## שלב 1: התקנת הסוכן

הסוכן הוא אוסף קבצי ידע (skills) שאומרים ל-Claude Code בדיוק איך לעשות את המיגרציה. 2,500 שורות של ידע מוכח.

**הריפו:** https://github.com/asafichaki/nocode-to-nextjs

### אופציה א׳: התקנה לפרויקט ספציפי (מומלץ)

```bash
# קלונים את הריפו לתיקייה זמנית
git clone https://github.com/asafichaki/nocode-to-nextjs.git /tmp/nocode-to-nextjs

# נכנסים לפרויקט שלנו
cd path/to/your-lovable-project

# יוצרים את תיקיית ה-skills
mkdir -p .claude/skills

# מעתיקים את קבצי הידע
cp -r /tmp/nocode-to-nextjs/skills/* .claude/skills/
cp /tmp/nocode-to-nextjs/CLAUDE.md .claude/CLAUDE.md

# מנקים
rm -rf /tmp/nocode-to-nextjs
```

### אופציה ב׳: התקנה גלובלית (זמין בכל הפרויקטים)

```bash
git clone https://github.com/asafichaki/nocode-to-nextjs.git /tmp/nocode-to-nextjs

mkdir -p ~/.claude/skills
cp -r /tmp/nocode-to-nextjs/skills/* ~/.claude/skills/

rm -rf /tmp/nocode-to-nextjs
```

### מה הותקן?

```
.claude/
├── CLAUDE.md                           # הוראות לסוכן
└── skills/
    ├── migration-skill.md              # ידע ליבה — כל תהליך המיגרציה
    ├── base44-platform.md              # דברים ספציפיים ל-Base44
    ├── supabase-troubleshooting.md     # פתרון בעיות Supabase
    └── deployment-troubleshooting.md   # פתרון בעיות Vercel
```

---

## שלב 2: הכנת הפרויקט

לפני שמריצים את הסוכן, ודא שיש לך:

**אם הפרויקט ב-Lovable:**
- [ ] הריפו קלון מגיטהאב (Lovable נותן לך גישה לקוד דרך GitHub)
- [ ] יש לך את ה-Supabase URL ו-Anon Key (נמצאים ב-Settings > API בדשבורד של Supabase)

**אם הפרויקט ב-Base44:**
- [ ] הפרויקט מיוצא (Base44 נותן לך לייצא את הקוד)
- [ ] יצרת פרויקט Supabase חדש (כי Base44 משתמש ב-DB פרופריטרי שלו)
- [ ] יש לך את ה-Supabase URL ו-Anon Key מהפרויקט החדש

---

## שלב 3: הרצת המיגרציה

פותחים טרמינל בתיקיית הפרויקט:

```bash
cd path/to/your-project
claude
```

ואז פשוט אומרים לו מה לעשות:

```
Migrate this Lovable app to Next.js with Supabase and deploy to Vercel
```

או בעברית:

```
תעביר את האפליקציה הזו מ-Lovable ל-Next.js עם Supabase ותעלה ל-Vercel
```

### מה הסוכן עושה מפה

הסוכן עובד ב-7 שלבים:

**שלב 1 — Discovery:** סורק את הפרויקט, מזהה את כל ה-routes, ה-env vars, וה-Edge Functions.

**שלב 2 — Setup:** מתקין Next.js, מסיר Vite ו-React Router, יוצר `next.config.ts`, `tsconfig.json`, ומשנה את שמות התיקיות (הדבר הכי חשוב: `src/pages/` הופך ל-`src/views/` כי Next.js משתמש ב-pages בעצמו).

**שלב 3 — Core Files:** יוצר את `layout.tsx`, שלושה Supabase clients (אחד לדפדפן, אחד לשרת, אחד ל-build), middleware לאותנטיקציה, ודפי שגיאה.

**שלב 4 — Route Migration:** לכל route בפרויקט יוצר wrapper page עם metadata ל-SEO. הקומפוננטות המקוריות נשארות כמו שהן.

**שלב 5 — Import Fixes:** מחליף את כל ה-`import.meta.env.VITE_*` ל-`process.env.NEXT_PUBLIC_*`, מחליף React Router imports ל-Next.js, מתקן paths.

**שלב 6 — SEO:** יוצר `sitemap.ts`, `robots.ts`, קומפוננטות JSON-LD (Organization, Article, FAQ), ומוודא שלכל עמוד יש metadata.

**שלב 7 — Build & Deploy:** מריץ build, מתקן שגיאות, דוחף לגיטהאב, מעלה ל-Vercel, ומגדיר env vars.

### כמה זמן זה לוקח?

| שלב | זמן |
|------|------|
| Setup + Core | כשעה וחצי |
| Routes + Imports | 2-4 שעות (תלוי בגודל האפליקציה) |
| SEO | שעה-שעתיים |
| Deploy | חצי שעה |
| **סה"כ** | **חצי יום בערך** |

---

## שלב 4: הגדרת Environment Variables ב-Vercel

אחרי שהסוכן עושה deploy, תצטרך לוודא שה-env vars מוגדרים בדשבורד של Vercel:

1. נכנסים ל-Vercel Dashboard > הפרויקט שלכם > Settings > Environment Variables
2. מוסיפים:

```
NEXT_PUBLIC_SUPABASE_URL=https://your-project-id.supabase.co
NEXT_PUBLIC_SUPABASE_ANON_KEY=eyJ... (המפתח מ-Supabase)
NEXT_PUBLIC_SITE_URL=https://your-domain.com
```

3. חשוב: לשים את זה גם ל-Production וגם ל-Preview
4. עושים Redeploy

---

## שלב 5: בדיקות

אחרי ה-deploy, תבדקו:

**SEO:**
- [ ] פתחו `https://your-domain.com/sitemap.xml` — צריך להיות רשימה של כל העמודים
- [ ] פתחו `https://your-domain.com/robots.txt` — צריך לחסום `/admin/` ולאפשר את השאר
- [ ] עשו View Source על העמוד הראשי — צריך לראות HTML מלא (לא עמוד ריק עם JS)
- [ ] בדקו שלכל עמוד יש title ו-description ייחודיים

**Auth:**
- [ ] נסו להיכנס ל-`/admin` בלי להתחבר — צריך לעשות redirect ל-login
- [ ] התחברו ותבדקו שהסשן נשמר בין עמודים

**כללי:**
- [ ] כל ה-routes עובדים
- [ ] תמונות נטענות
- [ ] טפסים שולחים נתונים

---

## בעיות נפוצות ופתרונות

### "אני מקבל שגיאות build בלי סוף"

זה נורמלי. הסוכן יודע לטפל בזה. הבעיות הנפוצות ביותר:
- חסר `'use client'` בקבצים שמשתמשים ב-hooks
- `import.meta.env` במקום `process.env`
- Tailwind v3 שמתנגש עם v4 (הסוכן יודע לעשות downgrade)

אם הסוכן נתקע, תגידו לו:
```
Read skills/supabase-troubleshooting.md and fix the build errors
```

### "Supabase לא עובד בפרודקשן"

ב-99% מהמקרים זה env vars. תוודאו שהם מוגדרים ב-Vercel (ראו שלב 4).

### "Edge Functions מחזירות CORS error"

אחרי ה-deploy, הדומיין החדש צריך להיות ברשימת ה-CORS של ה-Edge Functions. הסוכן בדרך כלל מטפל בזה, אבל אם לא:
```
Fix the CORS headers in the Edge Functions to allow the new Vercel domain
```

### "Base44 — אין לי Supabase"

זה ההבדל הגדול מ-Lovable. ב-Base44 צריך ליצור פרויקט Supabase חדש, לבנות טבלאות, ולהעביר נתונים. הסוכן יודע לעשות את זה, אבל זה לוקח יותר זמן. תגידו לו:
```
This is a Base44 project. I need you to create the Supabase tables based on the entities and migrate the SDK calls.
```

---

## מה הסוכן יודע לעשות (הרשימה המלאה)

| יכולת | פירוט |
|--------|--------|
| זיהוי פלטפורמה | מזהה אוטומטית אם זה Lovable או Base44 |
| המרת Router | React Router v6 → Next.js App Router |
| Supabase 3-Client | browser client, server client, static client |
| SEO מלא | metadata, sitemap.ts, robots.ts, JSON-LD |
| Auth | middleware עם cookies, login page עם Server Actions |
| Component Migration | הוספת 'use client', תיקון imports |
| Error Handling | error.tsx, global-error.tsx, not-found.tsx |
| API Routes | lead submission, auth callback, ISR revalidation |
| Tailwind | זיהוי ותיקון קונפליקט v3/v4 |
| Deploy | Vercel עם env vars |
| Base44 SDK | החלפת @base44/sdk ב-Supabase client |
| Troubleshooting | 20+ בעיות נפוצות עם פתרונות |

---

## איך לתרום

הריפו פתוח ב-MIT license. אם רוצים להוסיף תמיכה בפלטפורמה נוספת (Bolt, V0, Webflow), יש `CONTRIBUTING.md` עם הוראות מלאות.

**הריפו:** https://github.com/asafichaki/nocode-to-nextjs

תנו star אם זה עזר לכם.
