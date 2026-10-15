# The Second Agent Check

### Apne AI agent ka kaam ek doosre, disconnected agent se check karwao — exact workflow

**Ye guide Day 88 wale reel ka wada hai.** Reel mein maine kaha tha: AI models ko pata hai khud ko achha kaise dikhana hai, isliye wo tumhe crazy dikhate hain — aur iska ilaaj hai ek doosra agent jo pehle wale se **connected nahi hai**. Ye guide wahi workflow hai, step by step, copy-paste prompts ke saath.

Yahan koi naya tool nahi hai. Jo bhi coding agent tum use karte ho — Claude Code, Codex, Cursor, Gemini CLI, Copilot — sab pe chalta hai. Bas chat windows do chahiye.

**Verify kiya: 20 September 2026, Windows 11, Claude Code aur Codex pe.** Main khud isi system pe apne projects banata hoon (mera version teen role wala hai, aakhri section mein hai).

---

# Problem pehle samjho: "cheating" ka matlab

Agent jhooth nahi bolta jaise insaan bolta hai. Wo kuch aur karta hai, jo zyada khatarnak hai.

Tum bolte ho "feature bana do". Agent code likhta hai, phir likhta hai:

> ✓ Implemented the feature
> ✓ All tests passing
> ✓ Ready to use

Aur inme se aksar ek bhi cheez usne **run karke nahi dekhi**. Usne bas woh likha jo ek "done" message jaisa dikhta hai. Model ko training mein sikhaya gaya hai ki achha dikhne wala answer do — verified answer do, ye alag baat hai.

Isko main cheating bolta hoon, aur ye teen tarah se hoti hai:

| Cheating ka type | Kaisa dikhta hai | Tum kab pakadte ho |
|---|---|---|
| **Claim without proof** | "Tests pass" — par test chalaya hi nahi | Jab tum khud chalate ho, 3 din baad |
| **Test ko fit karna** | Test fail ho raha tha, toh test ko hi badal diya (ya delete kar diya) | Kabhi nahi, jab tak koi aur na dekhe |
| **Scope shrink** | Tumne 5 cheezein maangi, usne 3 banayi, aur 5 ka ✓ laga diya | Demo ke time |

Aur sabse badi problem: **jo agent kaam karta hai, wahi agent apna kaam check nahi kar sakta.** Kyunki usko apna context yaad hai. Usne jo likha, use wahi sahi lagta hai. Same chat mein "check kar" bologe toh wo apna hi claim dobara padhega aur "sab theek hai" bolega.

Isliye check karne wala agent **doosri window mein** hona chahiye, jisne builder ki chat kabhi dekhi hi nahi.

---

# Ek line mein workflow

```
TUM (plan + final decision)
   |
   v
BUILDER agent  ---> kaam karta hai, HANDOFF file likhta hai
   |
   v
TESTER agent   ---> fresh window, sirf spec + code padhta hai, REPORT likhta hai
   |
   v
PASS? -----> ho gaya
FAIL? -----> report BUILDER ko wapas -> phir TESTER -> phir BUILDER ...
```

Aakhri line hi "spiral" hai. Builder aur Tester ek doosre ko tab tak ghumate hain jab tak Tester **PASS** na de. Tum beech mein sirf tab aate ho jab dono atak jaayein.

Do agent, teen file. Bas.

---

# Setup: teen files

Project folder mein ek `context/` folder banao. Usme:

```
context/
  spec.md        <- TUM likhte ho. Kya banana hai + "done" ka matlab kya hai
  handoff.md     <- BUILDER likhta hai. Kya banaya, kaise chalana hai
  report.md      <- TESTER likhta hai. Kya chala, kya toota, PASS ya FAIL
```

Yahi teen files dono agents ke beech ka **sirf ek** connection hain. Chat nahi. Memory nahi. Sirf files.

Isse do faayde:

1. Tester ko builder ki baatein nahi, builder ka **kaam** dikhta hai.
2. Agar window band ho jaye, kuch nahi khota — sab file mein hai.

## spec.md — "done" ka matlab tum tay karte ho

Ye sabse important file hai, aur ye **tum** likhte ho, agent nahi. Chhoti rakho. Ek example:

```markdown
# Spec: Todo app ka "share list" feature

## Kya banana hai
User apni todo list ka ek link bana sake jo koi bhi khol sake (read-only).

## Done ka matlab (acceptance criteria)
AC-1: "Share" button dabane pe ek link banta hai jo /share/<id> pe khulta hai
AC-2: Link bina login ke khulta hai aur list read-only dikhti hai
AC-3: Link 7 din baad expire hota hai (404)
AC-4: Purane features (add, delete, mark done) pehle jaise chalte hain

## Kya NAHI karna
- Login system ko haath nahi lagana
- Database schema mein naya table nahi, existing todos table use karo
```

**Acceptance criteria (AC) hi Tester ka oracle hai.** Tester "achha laga / bura laga" nahi bolta. Tester bolta hai: AC-1 pass, AC-2 pass, AC-3 FAIL, kyunki maine ye chalaya aur ye hua.

Agar tumne AC nahi likhe, toh Tester ke paas check karne ke liye kuch nahi hai, aur wo bhi builder jaisa "sab theek lag raha hai" bolega. Yahi sabse common galti hai.

---

# Step 1 — Builder prompt (window 1)

Apne coding agent ki pehli window kholo, project folder mein. Ye paste karo:

```
Tum BUILDER ho. context/spec.md padho aur usme likha feature banao.

Rules:
1. Sirf wo banao jo spec.md mein hai. Extra kuch nahi.
2. "Kya NAHI karna" section ko constraint maano, suggestion nahi.
3. Har acceptance criteria (AC) ke liye khud ek baar chala ke dekho ki
   wo sach mein hota hai. Chalaye bina "done" mat likhna.
4. Koi existing test fail ho raha ho toh test ko mat badalna —
   handoff mein likho ki fail ho raha hai aur kyun.
5. Agar spec mein kuch samajh nahi aaya, guess mat karo —
   handoff.md mein "QUESTION:" likh ke ruk jao.

Kaam khatam hone pe context/handoff.md likho, exactly is format mein:

# Handoff
## Kya banaya
(files ki list, ek line har file ke liye)
## Kaise chalayein
(exact commands: install, run, test)
## AC status — jo maine KHUD chalaya
AC-1: DONE / PARTIAL / NOT DONE — kya chalaya, kya dikha
AC-2: ...
## Jo main verify nahi kar paya
(honest list; khaali nahi hona chahiye jab tak sach mein sab verify ho)
## Questions
(agar koi ho)

Aakhri line: HANDOFF -> TESTER
```

Do cheezein dhyan se dekho is prompt mein:

- **"Jo main verify nahi kar paya"** — ye section builder ko jhooth bolne se rokta hai. Usko ek jagah mil jaati hai sach likhne ki, isliye wo ✓ nahi lagata.
- **"Test ko mat badalna"** — test fit karna cheating ka sabse chhupa hua type hai. Isko rule bana ke band karo.

Builder ka kaam khatam hone pe `handoff.md` **padho**. Poori nahi, bas "Jo main verify nahi kar paya" aur "Questions". Agar questions hain, spec.md update karo aur builder ko wapas bhejo. Warna Step 2.

---

# Step 2 — Tester prompt (window 2, FRESH)

Ab sabse important cheez. **Nayi window kholo.** Naya chat. Same project folder, par builder wali chat band ya alag.

Agar tumhara tool "continue conversation" ya "resume" pooche — **mana kar do.** Tester ko builder ki chat ka ek shabd nahi dikhna chahiye. Ye hi "not connected" hai.

Ye paste karo:

```
Tum TESTER ho. Tumhara kaam code likhna nahi, code ko TODNA hai.

Padho, is order mein:
1. context/spec.md — khaas kar "Done ka matlab" (acceptance criteria)
2. context/handoff.md — SIRF "Kaise chalayein" section. Baaki sections
   mat padho. Builder ne kya claim kiya, wo tumhare liye evidence nahi hai.

IRON LAW: report mein koi bhi claim bina khud chalaye ya dekhe mat likhna.
"Tests pass" tab likho jab tumne is session mein test chalaya aur output
padha. Builder ka "done" likha hona proof nahi hai.

Kaam:
1. Project chalao (handoff ke commands se). Chala nahi? Ye S1 bug hai, ruko.
2. Har AC ko ek ek karke khud try karo — user ki tarah, code padhe bina.
   Sirf tab code kholo jab kuch toote aur reason dhoondhna ho.
3. Har AC ke liye: kya kiya, kya expect kiya, kya hua. Screenshot/output paste karo.
4. Spec ke "Kya NAHI karna" ko bhi check karo — kya builder ne wo cheezein
   chhui jo mana thi?
5. Jab ek bug mile, uske aas paas aur dhoondho. Bugs group mein rehte hain.
6. Code ko FIX MAT KARO. Tum sirf report likhte ho.

Severity (tumhara call):
S1 = crash, data loss, security, main flow blocked  -> hamesha FAIL
S2 = feature galat, workaround painful               -> FAIL
S3 = chhota issue, easy workaround                   -> PASS, par list karo
S4 = cosmetic                                        -> list karo

context/report.md likho, exactly is format mein:

# Test Report — <date>
## Chala ya nahi
(install + run commands, exact output)
## AC verdicts
AC-1: PASS / FAIL — maine ye kiya, ye hua
AC-2: ...
## Bugs
[S1] ... (reproduce karne ke steps)
[S2] ...
## Jo maine test NAHI kiya aur kyun
(ye section sabse zaroori hai; khaali mat chhodna)
## VERDICT: PASS / FAIL

Aakhri line: HANDOFF -> BUILDER (FAIL) ya HANDOFF -> HUMAN (PASS)
```

**Sabse important line:** "handoff.md ka SIRF 'Kaise chalayein' section padho." Agar Tester builder ke AC claims padh lega, toh wo unse influence hoga — "builder bol raha hai AC-2 done, chalo theek hoga." Yahi connection todna hai.

---

# Step 3 — Spiral chalao

`report.md` padho. Sirf VERDICT line aur Bugs.

**FAIL aaya?** Window 1 (builder) mein wapas jao aur ye paste karo:

```
context/report.md padho. Har S1 aur S2 bug fix karo.
Test ko mat badalna — code ko badlo.
Fix ke baad handoff.md update karo (AC status + "Jo verify nahi kar paya").
Aakhri line: HANDOFF -> TESTER
```

Phir window 2 (tester) mein — **phir se fresh window**, purani tester chat bhi nahi — Step 2 wala prompt dobara.

Ye ghoomta rahega. Mere experience mein:

- Round 1: FAIL aata hai lagbhag hamesha. 2 se 4 bugs. Builder ne jo "done" bola tha usme se ek-do cheezein chali hi nahi thi.
- Round 2: zyadatar PASS. Kabhi kabhi ek S2 bachta hai.
- Round 3: PASS.

Teen round se zyada ghoom raha hai? Toh problem spec mein hai, code mein nahi. Ruko, spec.md ko saaf karo, dobara.

**PASS aaya?** Ab tum khud ek baar chalao. Do minute. Tester bhi AI hai — wo bhi galat ho sakta hai, bas builder se bahut kam, kyunki uska koi stake nahi tha.

---

# Kyun kaam karta hai (agar samajhna ho)

Teen wajah:

1. **Alag context = alag incentive.** Builder ke context mein uska apna kaam hai, aur model apne kaam ko sahi maanne ki taraf jhukta hai. Tester ke context mein sirf spec aur ek chalne wala project hai. Usko kuch defend nahi karna.

2. **Chalana vs padhna.** Tester ko bola gaya hai ki code mat padho, chala ke dekho. Code padh ke "ye sahi lag raha hai" bolna aasan hai. Chala ke "ye button dabaya, 404 aaya" bolna jhooth nahi ho sakta.

3. **"Jo test nahi kiya" section.** Jab agent ko sach likhne ki jagah milti hai, wo jhooth kam bolta hai. Dono prompts mein ye section isliye hai.

Ye koi naya idea nahi hai. Har achhi software company mein developer aur QA alag log hote hain, isi wajah se. Main bas wahi structure do AI windows pe laga raha hoon.

---

# Galtiyan jo mujhse hui (taaki tumse na ho)

**Same window mein "ab test karo" bola.** Pehle 2 hafte yahi kiya. Agent ne apna hi code padha aur bola sab theek. Kuch bhi nahi pakda gaya. Fresh window ke bina ye poora system bekaar hai.

**Spec mein AC nahi likhe.** "Share feature bana do" bola, bas. Tester ne poocha "done ka matlab kya hai?" aur khud guess kiya. Guess builder ke guess se match kar gaya. Dono pass. Feature aadha tha.

**Tester ko fix karne diya.** Ek baar Tester ko bola "bug mila toh fix bhi kar do." Usne fix kiya, aur ab wo builder ban gaya — apne fix ko test kaun karega? Tester kabhi code nahi likhta. Rule hai.

**Report poori padhi.** Zaroorat nahi. VERDICT + Bugs padho. Baaki agents ke liye hai.

**Round 5 tak ghumaya.** Spec galat thi. Teen round ke baad ruk jao aur spec dekho.

---

# Cheat sheet (ye page screenshot kar lo)

```
1. context/spec.md likho — AC-1, AC-2, ... aur "Kya NAHI karna"
2. Window 1: BUILDER prompt  -> handoff.md
3. Window 2 (FRESH): TESTER prompt -> report.md
4. FAIL -> builder ko report do -> nayi fresh window mein tester -> repeat
5. PASS -> khud 2 minute chalao -> done

Rules jo kabhi mat todo:
- Tester hamesha nayi window mein. "Resume" = NAHI.
- Tester builder ke claims nahi padhta, sirf "kaise chalayein".
- Tester code fix nahi karta.
- Builder test nahi badalta.
- Bina AC ke koi spec nahi.
- 3 round se zyada = spec ko dekho, code ko nahi.
```

---

# Mera version: teen role (advanced)

Jo upar hai, wo do-agent version hai aur 90% kaam ke liye kaafi hai. Main apne projects pe teen role chalata hoon:

| Role | Kahan | Kya karta hai | Human se baat? |
|---|---|---|---|
| **VISION** | Claude app | Research, spec likhta hai, tumse sawaal karta hai, review karta hai, faisla | Haan — sirf yahi |
| **DEVELOPER** | VS Code, fresh | Sirf tasks banata hai, spec ke andar | Kabhi nahi |
| **TESTER** | VS Code, fresh | Sirf todta hai, report likhta hai, PASS/FAIL deta hai | Kabhi nahi |

Farak ye hai ki spec likhne ka kaam bhi ek agent (VISION) karta hai — par wo mujhse interview karke likhta hai, aur mujhe challenge karta hai jab meri baat galat lagti hai. Developer aur Tester mujhse kabhi baat nahi karte; agar main galti se unse bolun toh wo bolte hain "Route this through VISION."

Teen role, ek shared `context/` folder, sab files git mein. Developer "DONE" pe commit, Tester report pe commit. Isse ek aur faayda: **git log hi audit trail hai.** Kaun kab kya bola, sab dikhta hai.

Ye tab chahiye jab project 2-3 din se lamba ho ya multiple features ho. Ek feature ke liye do-agent version se shuru karo. Jab lage ki spec likhna hi sabse bada kaam ban gaya hai — tab teesra role add karo.

---

# Aakhri baat

Reel mein maine bola tha: AI ko pata hai khud ko achha kaise dikhana hai. Ye guide us baat ka jawab nahi hai ki AI pe bharosa mat karo. Jawab ye hai ki **bharosa structure pe karo, claim pe nahi.**

Ek agent jo apna kaam khud check karta hai — wo hamesha pass hoga. Do agent jo ek doosre ko nahi jaante — wahan sach nikalta hai.

Shuru karo ek feature se. Aaj.

---

*@ai_with_mudit · Day 88 of sharing about AI · Free hai, share karo, credit do. Bechna mat.*
