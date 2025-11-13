
#### Repeater

מצוין לבדיקות נקודתיות
**תפקיד:** שליחת וריאציות ידניות של בקשות — בדיקה ידנית וניסויים נקודתיים.  
**מתי להשתמש:** בדיקות ידניות לדוגמאות ספציפיות: בדיקת XSS נקודתי, בדיקת פרמטר אחרי שינוי ידני, בדיקת header/encoded payloads.  
**טיפים:** אידיאלי אחרי שמצאת בקשה חשודה ב‑Proxy — תנסה לשנות פרמטרים, headers, body ולראות תגובה מידית.

#### Intruder

שימושי בשביל לבדוק מלא פקטות שנשלחות עם פרמטרים
ובנוסף לכך אפשר לחבר לו מאקרואים, כך שלמשל אם פקטה תלויה בtoken שפקטה לפניה אמורה לקבל אפשר לעשות כך שכל פעם תישלח הפקטה שאמורה להישלח ללפניה וה-token יוסנף ויתווסף לפקטה (בצורה נכונה ) שאותה אנו שולחים מלא.

**תפקיד:** fuzzing/automation על בקשה אחת — שליחה המונית של payloads לפרמטרים.  
**יכולות עיקריות:** סוגי התקפות: payload positions, payload sets, payload processing, payload generators, macros (Session handling) ושילוב עם payload lists ו‑encoders.  
**מתי להשתמש:** brute-force, parameter fuzzing, enumeration (username/password), SQLi blind (time-based), parameter injection, content-based fuzzing.  
**טיפים:** השתמש ב‑macros/session handling כשהבקשה דורשת token/CSRF שמתקבל קודם; השתמש ב־greps/grep‑match כדי לסנן תוצאות משמעותיות; עבור בדיקות רחבות — Intruder נוח, אך זהירות בעומס על השרת.

proxy-הסנפה

#### Repeater vs Intruder — מתי מה?

- Repeater = ניסוי ידני, תשאול נקודתי.
    
- Intruder = אוטומציה/פאזינג/הרצת רשימות payloads על הרבה משתנים או הרבה הערכים.



#### Sequencer


מצוין בשביל לבדוק את רמת האקראיות של פרמטריםן, למשל אם אנו מקבלים login token
נוכל לשלוח את הפקטה שמקבלת אותו אל ה-sequencer והוא יפציץ וכל פעם ישמור את הערך של login token שאנו מקבלים חזרה ויתן לנו דוח.

**תפקיד:** ניתוח רמת האקראיות/אנטרופיה של טוקנים/סשנים.  
**יכולות עיקריות:** איסוף דגימות token, חישוב אנטרופיה, בדיקת דגם חוזר / ניבוי.  
**מתי להשתמש:** בדק אם cookies/session tokens/CSRF tokens / password reset tokens נראים פגיעים (ניבוי או חוזר).  
**טיפים:** שלח לדוגמה מבקשה שמייצרת token רבים (login repeated calls) ואסוף דגימות; אם האנטרופיה נמוכה — זה אדום.


#### Decoder
משק הצפנה ופענוח

**תפקיד:** כלי המרת נתונים — decode/encode, hashing, binary/hex/base64, smart decode (recursive). 
**מתי להשתמש:** כשצריך לפענח payloadים, פרמטרים ממוספרים, חיבור/שינוי של tokens מקודדים לפני שליחה, הכנה של payloadים שמתאימים לשרת.  
**טיפים:** שימוש ב‑Smart Decode כדי להסיר רב‑שכבות של קידוד; הכן payload ב‑Decoder ואז העתק ל‑Repeater/Intruder.


#### Comparer
ממשק השוואה, נוכל להשוות תשובות של פקטות באמצעותו.

**תפקיד:** השוואת שתי חתיכות נתונים (byte/ASCII) כדי לזהות הבדלים זעירים.  
**מתי להשתמש:** זיהוי שינויים קטנים בתגובת השרת (האם טוקן שונה אחרי בקשה X?), השוואת תגובות לפני/אחרי payload כדי למצוא data leakage.  
**טיפים:** מצוין ל־IDOR: השווה תכולות JSON בין בקשות שונות כדי למצוא הבדלים בפרטיות/הרשאות.


#### Organizer

**תפקיד:** שמירת בקשות/הערות/סידור workflow של בדיקות — אחסון snapshots של בקשות ותיעוד.  
**מתי להשתמש:** כשאתה בודק פרויקט ארוך ורוצה תיעוד וניהול ראיות.  
**טיפים:** תיעד שלבים חשובים בקצרה (מה נבדק, תוצאה, צעדי המשך).


#### Collaborator (Burp Collaborator)

**תפקיד:** בדיקות OOB — איתור blind SSRF, blind XSS, blind SQLi דרך קריאות DNS/HTTP לשרת collaborator.  
**מתי להשתמש:** כל תקיפה שדורשת תצפית חיצונית (server-side callbacks), כגון SSRF, blind deserialization, blind server-side template injection.  
**טיפים:** שלח payload שמכיל כתובת collaborator; בדוק אירועים בדשבורד Collaborator.


#### Spider (site crawling)

**תפקיד:** גילוי אוטומטי של קישורים/משאבים באתר.  
**מתי להשתמש:** מיפוי מהיר של האתר לפני בדיקות ידניות; איתור endpoints נסתרים.  
**טיפים:** השתמש ביחד עם Target map כדי לסמן אזורים מעניינים.

#### Scanner (Burp Professional)

**תפקיד:** סריקה אוטומטית לזיהוי חולשות יישום Web (XSS, SQLi, RCE וכו').  
**מתי להשתמש:** איסוף מהיר של בעיות נפוצות, coverage ראשוני. לא תחליף לבדיקות ידניות.  
**טיפים:** השאר פרק זמן לאחר סריקה להערכת false positives; השתמש konkretescope כדי למנוע סריקה על endpoints רגישים.

---

#### Extender (وExtensions / BApp Store)

**תפקיד:** להוסיף פונקציונליות — כלים נוספים (e.g., active scanner checks, custom payload generators, automations).  
**מתי להשתמש:** כשצריך פיצ'ר מיוחד שלא קיים ב‑Burp הבסיסי (e.g., SAML toolkit, JSON beautifier, heavy fuzzers).  
**טיפים:** חפש ב‑BApp Store הרחבות שמתאימות לצורך (OAST, SAML, SSO decoders וכו').

---

#### Session Handling / Macros

**תפקיד:** טיפול בסשנים/הרשאות אוטומטי — להריץ בקשת login/refresh לפני כל payload של Intruder/Replayer.  
**מתי להשתמש:** כשבקשה דורשת token דינמי (CSRF/session token) או שינוי state בין בקשות.  
**טיפים:** הגדר macros עבור flows מורכבים; וודא שאתה קורא ומעדכן token נכון עם regex.


#### דוגמת זרימה מומלצת לבדיקת endpoint חשוד

1. ליירט את ה‑flow ב‑Proxy ולמפות ב‑Target.
    
2. לשלוח ל‑Repeater ולבצע ניסויים ידניים (headers, cookies, params).
    
3. אם נדרש המון ערכים — להגדיר Intruder (positions + payload sets + macros אם צריך).
    
4. אם מדובר בטוקן/סשן — לשלוח דוגמאות ל‑Sequencer.
    
5. אם צריך decode/encode — להשתמש ב‑Decoder להכנה/פענוח payload.
    
6. להשוות תגובות עם Comparer כדי לזהות דליפות זעירות.
    
7. אם מתקבל callback לא ברור — השתמש ב‑Collaborator.
    
8. תעד הכל ב‑Organizer / Logger.





#### טבלת קיצור — איזו חולשה -> איזה מודול (חוקי אצבע)

- XSS (reflected/stored) — Proxy (ליירט) → Repeater (בדיקות ידניות) → Intruder (רשימות payloads) → Scanner (בדיקה אוטומטית)
    
- CSRF / token issues — Proxy → Repeater → Sequencer (לבדיקת אקראיות) → Session Handling/Macros
    
- SQL Injection (error / blind / time-based) — Proxy → Repeater (manual) → Intruder (payload lists / time-based) → Scanner
    
- IDOR / Authorization flaws — Proxy → Repeater → Comparer (השוואת תגובות) → Intruder (enumeration של IDs)
    
- SSRF / OOB vulnerability — Proxy → Repeater/Intruder → Collaborator (לבדיקת callbacks)
    
- Session fixation / session prediction — Proxy → Sequencer → Comparer
    
- File upload / RCE / deserialization — Proxy → Repeater → Collaborator (אם צריך OOB) → Scanner/Extender (כלי עזר)
    
- Authentication brute-force / credential stuffing — Intruder (עם רשימות) + Session Handling
    
- Parameter tampering / business logic bypass — Proxy → Repeater → Intruder (fuzzing) → Comparer
    
- Encryption/encoding obfuscation (JWT/SAML/hex/base64) — Decoder → Repeater → Extender (כלים מיוחדים ל‑SAML/JWT)
    
- Blind vulnerabilities (no immediate response) — Intruder (timing) / Collaborator (OOB) / Sequencer (token patterns)
    
- Response-diff subtle data-leak — Comparer → Decoder → Repeater
    

---


    
#### טיפים פרקטיים מהשטח

- תמיד שמור scope והגדר גבולות לפני הרצת Intruder/Scanner.  
    
- השתמש ב‑macros/session handling כשקיימים tokens דינמיים — זה יחסוך לך false negatives. 

- ה-sequencer חשוב - אפילו טווח קטן של אנטרופיה יכול להעיד על token ניתנים לניבוי.

 - ה- Comparer חסכוני בזמן— השוואת תגובות אוטומטית מראה הבדלים שלא נראים לעין.
    
