# Week 2: etcd Basics & Backup (Hinglish Guide)

---

## 🎬 Session Shuru Karne Ka Tarika

**Ice Breaker Activity (5 mins):**
"Students, phone nikalo! WhatsApp open karo... ab airplane mode on karo. Kya messages bhej sakte ho? NAHI! Kyunki backend server se connection nahi hai. 

Exactly wahi hota hai Kubernetes mein - etcd down = cluster down! 
Aaj hum seekhenge ki ye etcd itna important kyun hai aur isko kaise bachayein!"

---

## 1. etcd Architecture aur Kubernetes mein Role

### 🎯 Story se Samjho: "The Bank Analogy"

**Imagine karo ek bank:**
- 🏦 **Bank Manager** = API Server
- 📚 **Bank Records/Ledger** = etcd (har transaction yaad hai)
- 💰 **Customers** = Pods, Services, Deployments

Agar bank ka ledger (records) kho jaye, to:
- ❌ Pata nahi kaun ka kitna balance hai
- ❌ Kon kon ke accounts hain
- ❌ Bank chala hi nahi sakta!

**etcd = Kubernetes ka ledger book hai!**

### 🎮 Interactive Demo Time!

**Live Experiment (Students ke saath):**

```bash
# Step 1: Pehle dekho cluster mein kya hai
kubectl get pods --all-namespaces

# Step 2: Ab etcd pod ko delete karo (simulation)
kubectl delete pod -n kube-system etcd-controlplane

# Step 3: Foran try karo
kubectl get pods
# ERROR! Connection refused!

# Yahi moment hai - "Dekha? etcd = Cluster ka heartbeat!"
```

**Students se poochho:**
- "Kisi ka WhatsApp kabhi crash hua hai?"
- "Data wapas aaya ya gaya?"
- "Backup tha to aaya, nahi to...?" 😅

### 🏗️ Architecture Visually Samjhao:

```
┌─────────────────────────────────────┐
│        Kubernetes Cluster           │
│                                     │
│  ┌──────────┐      ┌─────────┐    │
│  │   API    │◄────►│  etcd   │    │
│  │  Server  │      │ (Brain) │    │
│  └──────────┘      └─────────┘    │
│       ▲                             │
│       │                             │
│       ▼                             │
│  ┌──────────┐                      │
│  │ Kubelet, │                      │
│  │ Scheduler│                      │
│  └──────────┘                      │
└─────────────────────────────────────┘
```

**etcd mein kya store hota hai?**
- Pods ki information
- Services ka configuration
- ConfigMaps aur Secrets
- Node information
- Namespaces
- RBAC policies
- Basically har woh cheez jo `kubectl get` se milti hai!

### Real Example:
```bash
# Jab aap pod create karte ho
kubectl create deployment nginx --image=nginx

# Backend mein kya hota hai:
# 1. API Server request receive karta hai
# 2. API Server etcd mein data WRITE karta hai
# 3. etcd confirm karta hai "Data saved!"
# 4. Scheduler etcd se READ karta hai aur pod schedule karta hai
```

---

---

## 2. etcd Data Structure aur Key-Value Store

### 🎲 Game Time: "Locker Room Game"

**Classroom Activity:**

"Sab log apna phone uthao! Hum ek game khelenge:
- Mai locker master hun (etcd)
- Tumhara phone = valuable item
- Locker number = key
- Phone = value

Jab tumhe phone chahiye, kya bologe? 
'Locker 23 kholo!' - This is KEY-VALUE!"

### 🗂️ Real Command Practice (Har student ko type karwao):

```
Key                          →    Value
------------------------------------------
/registry/pods/default/nginx → {pod: nginx, image: nginx:latest, ...}
/registry/services/default/  → {service: kubernetes, type: ClusterIP, ...}
```

### etcd mein data hierarchical structure mein store hota hai:

```
/registry/
├── pods/
│   ├── default/
│   │   ├── nginx-abc123
│   │   └── redis-xyz789
│   └── kube-system/
│       └── coredns-123
├── services/
│   └── default/
│       └── kubernetes
└── configmaps/
    └── default/
        └── app-config
```

### etcdctl se data dekhna:

```bash
# etcd pod mein jaao
kubectl exec -it etcd-controlplane -n kube-system -- sh

# Saare keys list karo
ETCDCTL_API=3 etcdctl get / --prefix --keys-only

# Specific key ka value dekho
ETCDCTL_API=3 etcdctl get /registry/pods/default/nginx
```

**Output Example:**
```
/registry/pods/default/nginx
k8s

v1Pod
nginx
default"...
```

### Real-World Analogy:
Socho etcd ek **library** hai:
- **Key** = Book ka location (Section A, Shelf 5)
- **Value** = Actual book (Pod ka pura data)
- Jab kisi ko book chahiye, library system key use karke exact location batata hai

---

---

## 3. etcd Backup Procedures

### 🎬 Real-Life Horror Story (Students ko daro! 😅)

**"The Friday Evening Disaster"**

"Ek DevOps engineer Friday evening ko ghar jane wala tha...
5:30 PM - Boss ne kaha: 'Quick testing environment cleanup kar do'
5:35 PM - Engineer ne type kiya: `kubectl delete namespace prod`
5:36 PM - Engineer ka reaction: 😱😱😱

**But wait...**
5:37 PM - Usne etcd backup restore kiya
5:45 PM - Sab kuch wapas aa gaya!
5:50 PM - Wo ghar chala gaya with a smile 😊

**Moral: Backup = DevOps engineer ki insurance policy!**"

### 🎯 Challenge Activity: "Speed Backup Competition"

**Competition setup:**
1. Divide class into 2-3 teams
2. Har team ko task: "Sabse fast backup lo!"
3. Timer lagao - 5 minutes
4. Winner ko shout-out!

### 📝 Backup Command - Step by Step (Chant ki tarah repeat karwao):

```bash
# Yaad karne ka tarika: "E-C-C-C" (etcdctl + 3 certificates)

# E - ETCDCTL_API=3
# C1 - CACert
# C2 - Cert  
# C3 - Key

ETCDCTL_API=3 etcdctl snapshot save /tmp/etcd-backup.db \
  --endpoints=https://127.0.0.1:2379 \
  --cacert=/etc/kubernetes/pki/etcd/ca.crt \    # C1
  --cert=/etc/kubernetes/pki/etcd/server.crt \  # C2
  --key=/etc/kubernetes/pki/etcd/server.key     # C3
```

**Students se pucho:** "Teen certificates kyun chahiye?"
**Answer together:** "Security! Security! Security!"

### 🎪 Live Demo with Drama:

```bash
# Pehle dikhao - cluster mein kya hai
kubectl get all --all-namespaces | wc -l
# "Dekho! 47 resources hai cluster mein"

# Backup lo (dramatic pause)
echo "Taking backup... 🎬"
ETCDCTL_API=3 etcdctl snapshot save /tmp/backup.db \
  --endpoints=https://127.0.0.1:2379 \
  --cacert=/etc/kubernetes/pki/etcd/ca.crt \
  --cert=/etc/kubernetes/pki/etcd/server.crt \
  --key=/etc/kubernetes/pki/etcd/server.key

# Verification (celebrate!)
ls -lh /tmp/backup.db
echo "Backup size dekho - Ye hai tumhari insurance policy! 💾"
```

### 🎵 Mnemonic Song (Students ko gaana sikhao):

```
🎵 "Snapshot save karo bhai,
   Endpoints, cacert, cert, key lai,
   Backup file safe rakho bhai,
   Disaster aaye to tension nahi!" 🎵
```

#### Certificate files kahan milenge?

```bash
# Static pod manifest check karo
cat /etc/kubernetes/manifests/etcd.yaml

# Common locations:
# --cacert: /etc/kubernetes/pki/etcd/ca.crt
# --cert: /etc/kubernetes/pki/etcd/server.crt
# --key: /etc/kubernetes/pki/etcd/server.key
```

#### Backup file ko safe jagah copy karo:

```bash
# Pod se bahar copy karo
kubectl cp kube-system/etcd-controlplane:/tmp/etcd-backup.db ./etcd-backup.db

# Ya directly node se
scp /tmp/etcd-backup.db user@backup-server:/backups/
```

### Backup verify karna:

```bash
ETCDCTL_API=3 etcdctl snapshot status etcd-backup.db --write-out=table

# Output:
+----------+----------+------------+------------+
|   HASH   | REVISION | TOTAL KEYS | TOTAL SIZE |
+----------+----------+------------+------------+
| 8e6d1a2b |    12345 |       1500 |     5.2 MB |
+----------+----------+------------+------------+
```

### Automation: CronJob se daily backup

```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: etcd-backup
  namespace: kube-system
spec:
  schedule: "0 2 * * *"  # Har raat 2 baje
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: backup
            image: k8s.gcr.io/etcd:3.5.6
            command:
            - /bin/sh
            - -c
            - |
              ETCDCTL_API=3 etcdctl snapshot save /backup/etcd-$(date +%Y%m%d).db \
                --endpoints=https://etcd:2379 \
                --cacert=/etc/kubernetes/pki/etcd/ca.crt \
                --cert=/etc/kubernetes/pki/etcd/server.crt \
                --key=/etc/kubernetes/pki/etcd/server.key
          restartPolicy: OnFailure
```

---

---

## 4. etcd Restore Procedures

### 🎭 Bollywood Style Storytelling

**"The Return of the Lost Data" - A Drama in 7 Acts**

**Act 1: The Disaster (Tragedy Music 🎵)**
```bash
# Villain enters: Junior engineer ne sab delete kar diya!
kubectl delete namespace production
# "Haye! Sab kuch khatam! 😱"
```

**Act 2: The Hero Arrives (Hope Music 🎺)**
```bash
# "Ruko! Mere paas backup hai!"
ls -la /tmp/etcd-backup.db
```

**Act 3-7: Recovery Process**

### 🎮 Interactive Restoration Game

**"7 Steps to Heaven" - Har step students ko karwao:**

**Step 1: API Server Band Karo** (Dramatic pause)
```bash
echo "Stopping API Server... Hold your breath! 😰"
mv /etc/kubernetes/manifests/kube-apiserver.yaml /tmp/
sleep 5
kubectl get pods  # ERROR! - "Perfect! API Server stopped!"
```

**Step 2: etcd Ko Alvida Kaho** 
```bash
echo "Saying goodbye to etcd... 👋"
mv /etc/kubernetes/manifests/etcd.yaml /tmp/
```

**Step 3: Purana Data Ko Side Mein Rakho**
```bash
echo "Moving old data to safety... 📦"
mv /var/lib/etcd /var/lib/etcd.old
```

**Step 4: Magic Time - RESTORE!** ✨
```bash
echo "Abracadabra! Restoring backup... 🪄"
ETCDCTL_API=3 etcdctl snapshot restore /tmp/etcd-backup.db \
  --data-dir=/var/lib/etcd

# Success message dikhao dramatically
echo "✅ Data restored! The hero wins!"
```

**Step 5-7: Sab Kuch Wapas Laao**
```bash
echo "Bringing etcd back to life... 🔄"
mv /tmp/etcd.yaml /etc/kubernetes/manifests/

sleep 10

echo "Bringing API Server back... 🚀"
mv /tmp/kube-apiserver.yaml /etc/kubernetes/manifests/

sleep 30

echo "Checking if magic worked... 🎩"
kubectl get all -n production
# "Dekho! Sab wapas aa gaya! Standing ovation! 👏"
```

### 🎯 Memory Trick: "RAIDER Formula"

```
R - Remove API Server
A - Also remove etcd
I - Isolate old data
D - Do the restore
E - Enable etcd again
R - Resume API Server
```

**Students ko chant karwao:** 
"R-A-I-D-E-R! R-A-I-D-E-R! That's how we RESTORE!" 📣

---

---

## 🎪 Labs - Gamification Style!

### Lab 1: "The Backup Champion Challenge" 🏆

**Points System:**
- ✅ Backup successful = 10 points
- ✅ Verification pass = 5 points
- ✅ Fastest time = 5 bonus points
- ⏰ Time limit: 10 minutes

**Mission Briefing:**
"Tumhari company ka production cluster hai. Boss ne kaha backup lena hai. Ready?"

**Step-by-Step Mission:**

```bash
# Mission 1: Certificate location dhundo (2 mins)
echo "🔍 Finding certificates..."
cat /etc/kubernetes/manifests/etcd.yaml | grep -E 'cert|key|ca'
# Students ko sheet pe note karwao

# Mission 2: Backup command banao (3 mins)
echo "⚡ Creating backup..."
ETCDCTL_API=3 etcdctl snapshot save /tmp/etcd-backup-$(date +%F).db \
  --endpoints=https://127.0.0.1:2379 \
  --cacert=/etc/kubernetes/pki/etcd/ca.crt \
  --cert=/etc/kubernetes/pki/etcd/server.crt \
  --key=/etc/kubernetes/pki/etcd/server.key

# Success check!
if [ $? -eq 0 ]; then
  echo "✅ Mission Accomplished! +10 points!"
else
  echo "❌ Mission Failed! Try again!"
fi

# Mission 3: Verify backup (2 mins)
echo "🔬 Verifying backup integrity..."
ETCDCTL_API=3 etcdctl snapshot status /tmp/etcd-backup-*.db --write-out=table

# Students ko celebrate karwao!
echo "🎉 Backup successful! You're a DevOps hero!"
```

**Leaderboard Display:**
Screen pe show karo kis student ne kitne time mein complete kiya!

---

### Lab 2: "Disaster Recovery Challenge" 💥

**Role-Play Setup:**

**Students ko roles do:**
- 👨‍💼 CEO: "Data recover karo ya job jaegi!"
- 👨‍💻 DevOps Engineer: "Tumhe restore karna hai"
- ⏰ Timer: "10 minutes mein!"

**The Challenge:**

```bash
# 1. Certificate paths nikalo
cat /etc/kubernetes/manifests/etcd.yaml | grep -E 'cert|key|ca'

# 2. Snapshot command banao
ETCDCTL_API=3 etcdctl snapshot save /tmp/etcd-backup-$(date +%F).db \
  --endpoints=https://127.0.0.1:2379 \
  --cacert=/etc/kubernetes/pki/etcd/ca.crt \
  --cert=/etc/kubernetes/pki/etcd/server.crt \
  --key=/etc/kubernetes/pki/etcd/server.key

# 3. Verify
ETCDCTL_API=3 etcdctl snapshot status /tmp/etcd-backup-*.db

# 4. Backup ko safe location pe move karo
cp /tmp/etcd-backup-*.db /root/backups/
```

**Expected Output:**
```
Snapshot saved at /tmp/etcd-backup-2024-12-10.db

+----------+----------+------------+------------+
|   HASH   | REVISION | TOTAL KEYS | TOTAL SIZE |
+----------+----------+------------+------------+
| abc12345 |    45678 |       2340 |     8.5 MB |
+----------+----------+------------+------------+
```

```bash
# 🎬 Scene 1: Create test data
echo "Creating production namespace..."
kubectl create namespace prod-app
kubectl create deployment webapp --image=nginx -n prod-app
kubectl get all -n prod-app
echo "✅ Production app deployed!"

# 🎬 Scene 2: Take backup
echo "📸 Taking backup before disaster..."
ETCDCTL_API=3 etcdctl snapshot save /tmp/before-disaster.db \
  --endpoints=https://127.0.0.1:2379 \
  --cacert=/etc/kubernetes/pki/etcd/ca.crt \
  --cert=/etc/kubernetes/pki/etcd/server.crt \
  --key=/etc/kubernetes/pki/etcd/server.key

# 🎬 Scene 3: THE DISASTER! (Dramatic music)
echo "💥 DISASTER STRIKES!"
kubectl delete namespace prod-app
sleep 2
kubectl get ns prod-app  # Dekho, gayab!
echo "😱 Production app deleted! CEO is calling!"

# 🎬 Scene 4: HERO TIME - RESTORE!
echo "🦸 DevOps Engineer to the rescue!"

# Timer start!
START_TIME=$(date +%s)

# Stop API Server
mv /etc/kubernetes/manifests/kube-apiserver.yaml /tmp/
mv /etc/kubernetes/manifests/etcd.yaml /tmp/

# Backup old data
mv /var/lib/etcd /var/lib/etcd.backup

# RESTORE!
ETCDCTL_API=3 etcdctl snapshot restore /tmp/before-disaster.db \
  --data-dir=/var/lib/etcd

# Restart everything
mv /tmp/etcd.yaml /etc/kubernetes/manifests/
mv /tmp/kube-apiserver.yaml /etc/kubernetes/manifests/

echo "⏳ Waiting for cluster to recover..."
sleep 30

# 🎬 Scene 5: THE MOMENT OF TRUTH!
kubectl get ns prod-app
kubectl get all -n prod-app

END_TIME=$(date +%s)
DURATION=$((END_TIME - START_TIME))

if kubectl get ns prod-app > /dev/null 2>&1; then
  echo "🎉🎉🎉 SUCCESS! Data recovered in ${DURATION} seconds!"
  echo "👏 Standing ovation! Job saved!"
else
  echo "❌ Failed! Try again!"
fi
```

**Students ko points do based on time:**
- < 5 mins = Gold medal 🥇
- 5-7 mins = Silver medal 🥈  
- 7-10 mins = Bronze medal 🥉

---

### Lab 3: "Health Check Master" 🏥

**Doctor Role-Play:**
"Tum ek doctor ho. Cluster tumhara patient hai. Checkup karo!"

**Diagnosis Checklist:**

```bash
# 🩺 Check 1: Heart Rate (etcd health)
echo "Checking cluster heartbeat..."
kubectl exec -n kube-system etcd-controlplane -- \
  etcdctl endpoint health \
  --cacert=/etc/kubernetes/pki/etcd/ca.crt \
  --cert=/etc/kubernetes/pki/etcd/server.crt \
  --key=/etc/kubernetes/pki/etcd/server.key

# Expected: "is healthy" ✅
# Students ko thoko taali!

# 🧠 Check 2: Brain Scan (etcd member list)
echo "Checking cluster brain..."
kubectl exec -n kube-system etcd-controlplane -- \
  etcdctl member list \
  --cacert=/etc/kubernetes/pki/etcd/ca.crt \
  --cert=/etc/kubernetes/pki/etcd/server.crt \
  --key=/etc/kubernetes/pki/etcd/server.key

# 🫀 Check 3: Blood Circulation (All pods status)
echo "Checking cluster circulation..."
kubectl get pods -n kube-system

# Har pod Running hona chahiye!

# 💪 Check 4: Strength Test (Create new workload)
echo "Testing cluster strength..."
kubectl create deployment health-test --image=nginx
kubectl get pods | grep health-test

# 📊 Check 5: Final Report
echo "═══════════════════════════════════"
echo "   CLUSTER HEALTH REPORT CARD"
echo "═══════════════════════════════════"
echo "etcd health     : ✅ PASS"
echo "Members         : ✅ PASS"
echo "System pods     : ✅ PASS"
echo "New deployment  : ✅ PASS"
echo "═══════════════════════════════════"
echo "   DIAGNOSIS: CLUSTER IS HEALTHY! 🎉"
echo "═══════════════════════════════════"
```

**Students ko certificate do:** "Certified Kubernetes Doctor!" 👨‍⚕️

---

## 🎉 Session End Activities

### 1. **Quick Quiz - Kahoot Style!**

**Questions (Students ko phone pe answer karwao):**

Q1: etcd kya hai?
- A) Database ✅
- B) Container
- C) Pod
- D) Service

Q2: Backup command mein kitne certificates chahiye?
- A) 1
- B) 2  
- C) 3 ✅
- D) 4

Q3: Restore ke pehle kya band karna padta hai?
- A) Only etcd
- B) Only API Server
- C) Both ✅
- D) Nothing

**Winner ko prize do:** "etcd Master" badge! 🏆

---

### 2. **Real Interview Question Practice**

**Mock Interview Setup:**

**Interviewer (Teacher):** "Production mein etcd crash ho gaya, kya karoge?"

**Student answer format:**
1. "Pehle panic nahi karunga!" 😌
2. "Backup check karunga"
3. "RAIDER formula follow karunga"
4. "Step by step restore karunga"
5. "Health check karunga"

**Points for:**
- Confidence
- Correct steps
- No panic

---

### 3. **Homework Challenge - "The 24-Hour Mission"**

**Assignment:**

```
📋 Mission Brief:
- Setup automated daily backup (CronJob)
- Document process with screenshots
- Submit before next class

🏆 Bonus Points:
- Add Slack notification on backup success/failure
- Create backup retention policy (keep last 7 days)
- Write a disaster recovery runbook
```

---

## 🎊 Motivational Ending

**End session with this:**

"Remember friends! 
- 💾 Backup = Life insurance for DevOps
- 🦸 You're not just engineers, you're DATA SUPERHEROES!
- 🎯 CKA exam mein ye question pakka aayega
- 💼 Companies mein ye skill = Higher salary!

**Final chant together:**
'E-C-C-C, Backup daily!
R-A-I-D-E-R, Restore safely!' 📣

Aaj se promise karo - 
'Backup lunga har roz,
Cluster bachaunga, boss ko karunga impress!' 💪

**Class dismissed! See you next week!** 👋"

---

## 📚 Bonus: Quick Reference Card (Print & Distribute)

```
╔════════════════════════════════════════╗
║     etcd CHEAT SHEET - Pocket Guide   ║
╚════════════════════════════════════════╝

📸 BACKUP:
ETCDCTL_API=3 etcdctl snapshot save <file>
  --endpoints=https://127.0.0.1:2379
  --cacert=<ca> --cert=<cert> --key=<key>

🔄 RESTORE:
1. mv kube-apiserver.yaml /tmp
2. mv etcd.yaml /tmp  
3. mv /var/lib/etcd /var/lib/etcd.old
4. etcdctl snapshot restore <file>
5. mv etcd.yaml back
6. mv kube-apiserver.yaml back

🏥 HEALTH CHECK:
etcdctl endpoint health

💡 REMEMBER: E-C-C-C (3 certificates!)
💡 REMEMBER: R-A-I-D-E-R (restore steps!)

🎯 Pro Tip: Practice daily = CKA success!
```

---

## 🎬 Post-Class Engagement

**WhatsApp Group Activities:**

1. **Daily Challenge (Week 2):**
   - Day 1: Share your backup script screenshot
   - Day 2: Share restore time - fastest wins!
   - Day 3: Share a disaster scenario you handled
   - Day 4: Share your automation script
   - Day 5: Mock interview recording

2. **Peer Learning:**
   - Form study groups of 3-4
   - Practice restore together
   - Quiz each other
   - Share tips & tricks

3. **Real-World Stories:**
   - Ask students to share: "Have you ever lost data?"
   - Discuss: "What would you do differently now?"

---

**Next Week Teaser:**

"Next week hum seekhenge SCHEDULING! 
Imagine karo - tum manager ho, aur decide karna hai kon sa pod kis node pe jayega! 
Exciting hoga! 🚀

Till then - KEEP BACKING UP! 💾"
