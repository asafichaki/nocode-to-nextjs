# איך להעביר את האפליקציה שלך מ-Lovable / Base44 לאתר מקצועי

## בשביל מי זה

בנית אפליקציה או אתר ב-Lovable או Base44. הכל נראה טוב, אבל גוגל לא מוצא אותך, האתר איטי, ואתה מרגיש שזה לא מספיק מקצועי לפרודקשן אמיתי.

הכלי הזה פותר את זה. הוא לוקח את מה שבנית ומעביר את זה לטכנולוגיה מקצועית (Next.js) — בלי שתצטרך לכתוב קוד בעצמך. סוכן AI עושה את העבודה.

---

## מה תקבל בסוף

- גוגל יראה ויאנדקס את כל העמודים שלך
- כל עמוד יקבל כותרת ותיאור ייחודיים לחיפוש
- האתר יעלה ב-Vercel (אירוח חינמי ומהיר)
- מערכת הרשאות מקצועית (אזור אדמין מוגן)
- הכל ימשיך לעבוד עם Supabase (בסיס הנתונים)

---

## מה צריך לפני שמתחילים

### 1. להתקין Node.js

זה כלי בסיסי שצריך כדי שהכל ירוץ. פותחים טרמינל (ב-Mac: חיפוש "Terminal") ומריצים:

```
node -v
```

אם יצא מספר גרסה — מעולה, זה מותקן. אם לא, תורידו מפה:
https://nodejs.org (תבחרו בגרסה LTS)

### 2. חשבונות (חינם)

- **Supabase** — https://supabase.com — אם בניתם ב-Lovable כבר יש לכם. אם בניתם ב-Base44, תצטרכו להירשם.
- **Vercel** — https://vercel.com — שם האתר יעלה לאוויר. הרשמה חינם.
- **Anthropic** — https://console.anthropic.com — שם מקבלים מפתח API ל-Claude (יש תקופת ניסיון).

### 3. להתקין את Claude Code

Claude Code זה בעצם מתכנת AI שעובד דרך הטרמינל. הוא קורא את הקוד שלכם, מבין מה צריך לעשות, ועושה את זה.

פותחים טרמינל ומריצים:

```
npm install -g @anthropic-ai/claude-code
```

אחרי ההתקנה, צריך להגדיר את המפתח שלכם:

1. נכנסים ל-https://console.anthropic.com
2. יוצרים חשבון
3. הולכים ל-API Keys ויוצרים מפתח
4. מריצים בטרמינל (תחליפו את your-key-here במפתח שקיבלתם):

```
export ANTHROPIC_API_KEY=your-key-here
```

ב-Mac, כדי שזה ישאר גם אחרי שתסגרו את הטרמינל:

```
echo 'export ANTHROPIC_API_KEY=your-key-here' >> ~/.zshrc
```

---

## בואו נתחיל

### צעד 1: מורידים את הסוכן

הסוכן הוא "מוח" שמלמד את Claude Code איך לעשות את ההעברה. הורדה פעם אחת ואתם מסודרים.

פותחים טרמינל ומריצים שורה אחרי שורה:

```
git clone https://github.com/asafichaki/nocode-to-nextjs.git /tmp/nocode-to-nextjs
```

### צעד 2: מתקינים את הסוכן בפרויקט שלכם

נכנסים לתיקיית הפרויקט שלכם (תחליפו את הנתיב):

```
cd path/to/your-project
```

ואז:

```
mkdir -p .claude/skills
cp -r /tmp/nocode-to-nextjs/skills/* .claude/skills/
cp /tmp/nocode-to-nextjs/CLAUDE.md .claude/CLAUDE.md
rm -rf /tmp/nocode-to-nextjs
```

זהו. הסוכן מותקן.

### צעד 3: מפעילים את Claude Code

עדיין בתיקיית הפרויקט, מריצים:

```
claude
```

ייפתח ממשק של Claude. עכשיו פשוט כותבים לו:

**אם הפרויקט מ-Lovable:**
```
Migrate this Lovable app to Next.js with Supabase and deploy to Vercel
```

**אם הפרויקט מ-Base44:**
```
Migrate this Base44 app to Next.js with Supabase and deploy to Vercel
```

### צעד 4: נותנים לו לעבוד

מפה הסוכן עובד לבד. הוא:

1. סורק את הפרויקט ומבין מה יש בו
2. מתקין את מה שצריך ומסיר את מה שלא
3. בונה את כל המבנה מחדש בצורה מקצועית
4. מוסיף SEO לכל עמוד (שגוגל ימצא אותכם)
5. בונה מערכת הרשאות
6. מתקן שגיאות (יהיו, זה נורמלי)
7. מעלה הכל ל-Vercel

הוא ישאל אתכם שאלות בדרך. פשוט ענו לו.

**כמה זמן זה לוקח?** בערך חצי יום. תלוי בגודל הפרויקט.

### צעד 5: בדיקה שהכל עובד

אחרי שהסוכן סיים, בדקו:

- [ ] נכנסים לכתובת שקיבלתם מ-Vercel — האתר עולה?
- [ ] מוסיפים `/sitemap.xml` לכתובת — רואים רשימת עמודים?
- [ ] לוחצים Ctrl+U (או Cmd+U ב-Mac) — רואים HTML מלא ולא עמוד ריק?
- [ ] כל הדפים שהיו באתר הישן עובדים גם בחדש?

---

## משהו לא עובד?

### הסוכן נתקע באמצע

תכתבו לו:
```
Continue with the migration, read skills/supabase-troubleshooting.md if needed
```

### האתר עלה אבל Supabase לא מתחבר

כנראה צריך להגדיר משתנים ב-Vercel. תכתבו לסוכן:
```
The Supabase connection is not working in production. Check the environment variables in Vercel.
```

### בניתם ב-Base44 ואין לכם Supabase

זה צפוי. ב-Base44 צריך ליצור בסיס נתונים חדש. תכתבו לסוכן:
```
This is a Base44 project. Create Supabase tables based on the entities and migrate everything.
```

### עדיין נתקעתם?

פתחו issue בגיטהאב ואני אעזור:
https://github.com/asafichaki/nocode-to-nextjs/issues

---

## סיכום — מה עשינו פה

| לפני | אחרי |
|-------|-------|
| גוגל לא רואה את האתר | גוגל רואה ומאנדקס הכל |
| אין כותרות ותיאורים לחיפוש | כל עמוד עם כותרת ותיאור |
| אין מפת אתר | מפת אתר אוטומטית |
| הכל רץ בדפדפן בלבד | רץ גם בשרת (מהיר יותר) |
| בסיס נתונים אחד פשוט | 3 חיבורים מותאמים |
| אזור אדמין בסיסי | אזור אדמין מוגן עם הרשאות |
| אירוח ב-Lovable/Base44 | אירוח מקצועי ב-Vercel |

---

**הריפו:** https://github.com/asafichaki/nocode-to-nextjs

עזר לכם? תנו כוכב בגיטהאב. נתקעתם? שלחו הודעה.
