দুই বছর আগে SkyNet Bangladesh শুরু করেছিল মাত্র দুটো পপ নিয়ে - মিরপুর আর কল্যাণপুর। প্রায় পাঁচ হাজার কাস্টমার। তখন তারা Nautobot সেটআপ করেছিল। সেই পরিকল্পনা অনুযায়ী কাজ করেছে, সফল হয়েছে। এখন তাদের গ্রোথ হয়েছে। বর্তমানে নর্থ ঢাকায় তাদের আটটা পপ, প্রায় পঞ্চাশ হাজার কাস্টমার। আর সবচেয়ে বড় কথা - তারা এখনও সেই একই Nautobot সিস্টেম ব্যবহার করছে যেটা দুই বছর আগে সেটআপ করেছিল।

কিন্তু স্কেল বাড়ার সাথে সাথে নতুন চ্যালেঞ্জ এসেছে। এই চ্যাপ্টারে আমরা দেখব কীভাবে SkyNet সেই চ্যালেঞ্জগুলো সামলেছে, কী কী পরিবর্তন করেছে, আর কীভাবে Nautobot তাদের এই স্কেলিং জার্নিতে সাহায্য করেছে।

### বর্তমান পরিস্থিতি - ২০২৬ সালের গল্প

আজ থেকে দুই বছর আগে (২০২৪) যেখানে ছিল:
- ২টা পপ
- ১৯টা ডিভাইস
- ৫ হাজার কাস্টমার

এখন ২০২৫ সালে কোথায় এসেছে:
- ৮টা পপ (নর্থ ঢাকা জুড়ে)
- ১২০+ ডিভাইস
- ৫০ হাজার কাস্টমার
- ১৫ জন টেকনিক্যাল টিম

### নতুন পপগুলো

মিরপুর আর কল্যাণপুরের পাশাপাশি নতুন ছয়টা পপ যুক্ত হয়েছে:

১. **উত্তরা পপ** - সেক্টর ৭, উত্তরা (~৮ হাজার কাস্টমার)
২. **বনানী পপ** - রোড ১১, বনানী (~৫ হাজার কাস্টমার)
৩. **গুলশান পপ** - গুলশান-২ (~৭ হাজার কাস্টমার)
৪. **মোহাম্মদপুর পপ** - টাউন হল রোড (~৬ হাজার কাস্টমার)
৫. **ধানমন্ডি পপ** - রোড ২৭, ধানমন্ডি (~৮ হাজার কাস্টমার)
৬. **বারিধারা পপ** - বারিধারা DOHS (~৪ হাজার কাস্টমার)

প্রতিটা পপে:

- ১টা কোর রাউটার
- ২-৩টা ডিস্ট্রিবিউশন সুইচ
- ১০-২০টা এক্সেস সুইচ

### নতুন আপস্ট্রিম কানেকশন

শুধু BTCL আর Summit না, এখন আরো যুক্ত হয়েছে:
- **ফাইবার@হোম** - ব্যাকআপ কানেকশন
- **সাইবারগেট** - কর্পোরেট ক্লায়েন্টদের জন্য ডেডিকেটেড
- **আন্তঃপপ লিংক** - নিজেদের ফাইবার দিয়ে পপ-টু-পপ কানেকশন

### প্রথম চ্যালেঞ্জ: স্ট্রাকচার রিঅর্গানাইজেশন

৫ হাজার কাস্টমারের সময় সব ফ্ল্যাট স্ট্রাকচারে রাখা যেত। কিন্তু ৫০ হাজারে সেটা কাজ করে না। দরকার হায়ারার্কিক্যাল স্ট্রাকচার।

### Location Hierarchy তৈরি করা

আগে যেখানে শুধু পপ ছিল, এখন একটা হায়ারার্কি তৈরি করা হয়েছে:

```
SkyNet Bangladesh
  └── Dhaka North Zone
       ├── Mirpur Cluster
       │    ├── Mirpur POP
       │    └── Kalyanpur POP
       ├── Uttara Cluster
       │    ├── Uttara POP
       │    └── Baridhara POP
       ├── Gulshan Cluster
       │    ├── Gulshan POP
       │    └── Banani POP
       └── Central Cluster
            ├── Dhanmondi POP
            └── Mohammadpur POP
```

এই স্ট্রাকচার Nautobot-এ কীভাবে তৈরি করবেন?

**Location Types তৈরি করুন:**

```
1. Zone
   - Name: Zone
   - Description: Geographic zone
   - Nestable: ✓

2. Cluster
   - Name: Cluster
   - Description: Group of nearby POPs
   - Nestable: ✓
   - Parent: Zone

3. POP
   - Name: POP
   - Description: Point of Presence
   - Nestable: ✗
   - Parent: Cluster
```

**এখন Location তৈরি করুন:**

Top Level:
```
Name: Dhaka North Zone
Location Type: Zone
Status: Active
Description: Northern Dhaka coverage area
```

Clusters:
```
Name: Mirpur Cluster
Location Type: Cluster
Parent: Dhaka North Zone
Description: Mirpur and Kalyanpur area coverage

Name: Uttara Cluster
Location Type: Cluster
Parent: Dhaka North Zone

Name: Gulshan Cluster
Location Type: Cluster
Parent: Dhaka North Zone

Name: Central Cluster
Location Type: Cluster
Parent: Dhaka North Zone
```

এখন প্রতিটা পপকে তার Cluster-এর আন্ডারে রাখুন:

```
Name: Mirpur POP
Location Type: POP
Parent: Mirpur Cluster

Name: Kalyanpur POP
Location Type: POP
Parent: Mirpur Cluster

Name: Uttara POP
Location Type: POP
Parent: Uttara Cluster
... এভাবে সবগুলো
```

এই হায়ারার্কি তৈরি হয়ে গেলে রিপোর্টিং অনেক সহজ হয়। যেমন "Mirpur Cluster-এ মোট কয়টা ডিভাইস?" - এক ক্লিকে উত্তর পাবেন।

### দ্বিতীয় চ্যালেঞ্জ: ডিভাইস নেমিং স্কেল করা

আগের নেমিং কনভেনশন ছিল:
```
[Type]-[Site]-[Role]-[Number]
উদাহরণ: R-MIR-CORE-01
```

এটা ভালো কাজ করছিল যখন দুটো সাইট ছিল। কিন্তু এখন আটটা সাইট, আর ভবিষ্যতে হয়তো বিশটা হবে। তাই নেমিং কনভেনশন আপডেট করা হয়েছে:

**নতুন কনভেনশন:**
```
[Type]-[Zone]-[Site]-[Role]-[Number]

Zone Codes:
  DN = Dhaka North
  DS = Dhaka South (ভবিষ্যতের জন্য)
  CT = Chittagong (ভবিষ্যতের জন্য)

Site Codes (3 letter):
  MIR = Mirpur
  KAL = Kalyanpur
  UTT = Uttara
  BAN = Banani
  GUL = Gulshan
  MOH = Mohammadpur
  DHA = Dhanmondi
  BAR = Baridhara

উদাহরণ:
  R-DN-MIR-CORE-01 = Router, Dhaka North, Mirpur, Core, #1
  R-DN-UTT-CORE-01 = Router, Dhaka North, Uttara, Core, #1
  SW-DN-GUL-ACC-15 = Switch, Dhaka North, Gulshan, Access, #15
```

কিন্তু সমস্যা হলো পুরোনো ডিভাইসগুলো আগের নেমিং-এ আছে। সেগুলো কী করবেন?

### বাল্ক রিনেম স্ট্রাটেজি

সব পুরোনো ডিভাইসকে একসাথে রিনেম করা দরকার। Nautobot UI থেকে একটা একটা করে করা সম্ভব না। এজন্য Python স্ক্রিপ্ট লিখতে হবে।

প্রথমে সব ডিভাইস Export করুন CSV-তে। Excel-এ ওপেন করুন। একটা নতুন কলাম `new_name` যোগ করুন। Formula দিয়ে নতুন নাম জেনারেট করুন:

```
Old: R-MIR-CORE-01
New: R-DN-MIR-CORE-01
```

তারপর এই CSV দিয়ে আপডেট করুন। অথবা একটা Python স্ক্রিপ্ট:

```python
from pynautobot import api

# Nautobot connection
nb = api(url="https://nautobot.skynet.bd", token="your-token")

# সব Mirpur ডিভাইস নিয়ে আসুন
devices = nb.dcim.devices.filter(location="Mirpur POP")

for device in devices:
    old_name = device.name
    # R-MIR-CORE-01 -> R-DN-MIR-CORE-01
    if old_name.startswith("R-MIR-"):
        new_name = old_name.replace("R-MIR-", "R-DN-MIR-")
    elif old_name.startswith("SW-MIR-"):
        new_name = old_name.replace("SW-MIR-", "SW-DN-MIR-")
    else:
        continue
    
    print(f"Renaming: {old_name} -> {new_name}")
    device.name = new_name
    device.save()
```

এভাবে সব সাইটের জন্য চালান। কিন্তু **সাবধান**: প্রথমে একটা ব্যাকআপ নিন। তারপর ছোট একটা ব্যাচে টেস্ট করুন।

### তৃতীয় চ্যালেঞ্জ: IP Address Management স্কেলিং

৫ হাজার কাস্টমারের সময় যে /22 পাবলিক ব্লক ছিল সেটা যথেষ্ট ছিল। কিন্তু ৫০ হাজারে সেটা শেষ হয়ে গেছে। নতুন আইপি ব্লক এসেছে।

### পুরোনো আইপি স্ট্রাকচার (২০২৪):

```
103.125.40.0/22 (1024 IPs)
  ├─ 103.125.40.0/24 - Residential Mirpur
  ├─ 103.125.41.0/24 - Residential Kalyanpur
  ├─ 103.125.42.0/25 - Corporate
  └─ 103.125.42.128/25 - Infrastructure
```

### নতুন আইপি স্ট্রাকচার (২০২৬):

আরো তিনটা /22 ব্লক এলোকেট হয়েছে:

```
103.125.40.0/22 (Original - প্রায় ফুল)
103.125.44.0/22 (New Block 1)
103.125.48.0/22 (New Block 2)
103.125.52.0/22 (New Block 3)
```

এখন এগুলোকে সুন্দর করে অর্গানাইজ করা হয়েছে:

```
Parent: 103.125.40.0/21 (Aggregate - সব ব্লক মিলে)
  ├─ 103.125.40.0/22 - Mirpur/Kalyanpur Cluster
  │    ├─ 103.125.40.0/24 - Mirpur Residential
  │    ├─ 103.125.41.0/24 - Kalyanpur Residential
  │    ├─ 103.125.42.0/24 - Mirpur Corporate
  │    └─ 103.125.43.0/24 - Infrastructure
  │
  ├─ 103.125.44.0/22 - Uttara/Baridhara Cluster
  │    ├─ 103.125.44.0/24 - Uttara Residential
  │    ├─ 103.125.45.0/24 - Baridhara Residential
  │    ├─ 103.125.46.0/24 - Uttara Corporate
  │    └─ 103.125.47.0/24 - Reserved
  │
  ├─ 103.125.48.0/22 - Gulshan/Banani Cluster
  │    └─ ... (similar breakdown)
  │
  └─ 103.125.52.0/22 - Central Cluster + Future
       └─ ... (similar breakdown)
```

এই স্ট্রাকচার Nautobot-এ ইমপ্লিমেন্ট করতে:

১. **Aggregate তৈরি করুন** (যদি Nautobot ভার্সন সাপোর্ট করে):

```
Prefix: 103.125.40.0/21
Type: Container
Description: SkyNet Bangladesh - All Public IP Blocks
```

২. **প্রতিটা /22 ব্লক:**

```
Prefix: 103.125.40.0/22
Parent: 103.125.40.0/21
Type: Container
Location: Mirpur Cluster
Description: Mirpur/Kalyanpur IP allocation
```

৩. **Child prefixes সাইট অনুযায়ী:**

```
Prefix: 103.125.40.0/24
Parent: 103.125.40.0/22
Type: Network
Location: Mirpur POP
VLAN: RESIDENTIAL_MIR
Description: Residential customers - Mirpur
```

### IP Utilization Tracking

৫০ হাজার কাস্টমারে আইপি ইউটিলাইজেশন ট্র্যাক করা অত্যন্ত জরুরি। Nautobot-এর Prefix লিস্টে গেলে দেখবেন প্রতিটা প্রিফিক্সের পাশে Utilization দেখাচ্ছে।

যেমন:
```
103.125.40.0/24 - 87% utilized (223/256 IPs used)
103.125.44.0/24 - 45% utilized (115/256 IPs used)
```

যখন কোনো প্রিফিক্স ৮০% এর বেশি ইউটিলাইজড হয়, তখন সতর্ক থাকুন। নতুন প্রিফিক্স প্ল্যান করুন।

### VLAN Scaling

আগে ভিল্যান স্ট্রাকচার ছিল সিম্পল:
```
VLAN 10: Management
VLAN 100: Residential
VLAN 200: Corporate
```

এখন প্রতিটা সাইটের জন্য আলাদা ভিল্যান:

```
VLAN 10-19: Management (site-specific)
  - VLAN 10: MGMT_MIRPUR
  - VLAN 11: MGMT_KALYANPUR
  - VLAN 12: MGMT_UTTARA
  ... ইত্যাদি

VLAN 100-199: Residential (site-specific)
  - VLAN 100: RES_MIRPUR
  - VLAN 101: RES_KALYANPUR
  - VLAN 102: RES_UTTARA
  ... ইত্যাদি

VLAN 200-299: Corporate (সব সাইট শেয়ারড)
  - VLAN 200: CORP_STANDARD
  - VLAN 201: CORP_PREMIUM
  - VLAN 202: CORP_ENTERPRISE
```

এই সব ভিল্যান Nautobot-এ CSV দিয়ে বাল্ক ইমপোর্ট করুন।

### চতুর্থ চ্যালেঞ্জ: Cable Management বাস্তবতা

৫ হাজার কাস্টমারের সময় সব ক্যাবল ডকুমেন্ট করা সম্ভব ছিল। কিন্তু ৫০ হাজারে এসে বোঝা গেল প্রতিটা প্যাচ কর্ড ট্র্যাক করা impractical।

### Pragmatic Cable Management

SkyNet একটা প্র্যাগম্যাটিক অ্যাপ্রোচ নিয়েছে:

**ডকুমেন্ট করবেন:**
- আপলিংক ক্যাবল (প্রোভাইডার টু কোর)
- কোর টু ডিস্ট্রিবিউশন লিংক
- ইন্টার-পপ ফাইবার লিংক
- ক্রিটিক্যাল ব্যাকআপ লিংক

**ডকুমেন্ট করবেন না:**
- এক্সেস সুইচ টু কাস্টমার প্যাচ কর্ড
- NOC-এর ইন্টারনাল প্যাচিং
- টেম্পোরারি টেস্ট ক্যাবল

এই পলিসি ডকুমেন্ট করে রাখুন:

```
SkyNet Cable Documentation Policy

Critical Cables (Must document):
1. Uplink cables from providers
2. Core to Distribution links
3. Inter-POP fiber links
4. Backup/redundancy links
5. Any cable >50m length

Non-Critical (Optional):
1. Access to customer patch cords
2. Internal NOC patching
3. Management connections
4. Temporary cables

Update Frequency:
- Critical: Immediate (same day)
- Non-Critical: Monthly audit
```

### পঞ্চম চ্যালেঞ্জ: Multi-Team Collaboration

৫ হাজার কাস্টমারের সময় তিনজন টেকনিশিয়ান ছিল। এখন ১৫ জন। বিভিন্ন টিম:

**NOC Team (৫ জন):**
- ২৪/৭ মনিটরিং
- ইনসিডেন্ট রেসপন্স
- কাস্টমার সাপোর্ট

**Field Operations (৬ জন):**
- নতুন কানেকশন ইনস্টলেশন
- ফল্ট রিপেয়ার
- সাইট মেইনটেন্যান্স

**Network Engineering (৩ জন):**
- নেটওয়ার্ক ডিজাইন
- কনফিগারেশন ম্যানেজমেন্ট
- ক্যাপাসিটি প্ল্যানিং

**Management (১ জন):**
- ওভারভিউ এবং রিপোর্টিং

### Permission Structure

এই টিমগুলোর জন্য আলাদা আলাদা পারমিশন:

**Group: NOC_Operators**
```
Permissions:
- View: সবকিছু
- Add/Edit: IP Addresses, Device Status
- No Delete permission
- No Provider/Circuit edit
```

**Group: Field_Technicians**
```
Permissions:
- View: Devices, Cables, IPs
- Add/Edit: Cables, Device Comments
- Add: Devices (নতুন এক্সেস সুইচ)
- No Delete, No IP allocation
```

**Group: Network_Engineers**
```
Permissions:
- View: সবকিছু
- Add/Edit/Delete: সব (Devices, IPs, Cables, Circuits)
- Cannot: Delete Locations, Delete Providers
```

**Group: Management_ReadOnly**
```
Permissions:
- View Only: সবকিছু
- Export: Reports
```

### Collaboration Workflow

একটা স্ট্যান্ডার্ড ওয়ার্কফ্লো:

**নতুন সাইট যুক্ত করার সময়:**

১. **Network Engineer** নতুন সাইট প্ল্যান করে:
   - Location তৈরি করে
   - IP allocation করে
   - Device order দেয়

২. **Field Team** সাইট সেটআপ করে:
   - ডিভাইস ইনস্টল করে
   - Serial numbers নোট করে
   - Cable routing করে

৩. **Network Engineer** Nautobot আপডেট করে:
   - Devices এড করে
   - Cables ডকুমেন্ট করে
   - IP assign করে

৪. **NOC Team** যাচাই করে:
   - সব ডিভাইস Nautobot-এ আছে কিনা
   - Monitoring সেটআপ হয়েছে কিনা

### ষষ্ঠ চ্যালেঞ্জ: Performance এবং Response Time

১২০+ ডিভাইস, ৮টা সাইট, হাজার হাজার আইপি - Nautobot UI কি স্লো হয়ে যাবে?

### Performance Optimization

SkyNet কিছু optimization করেছে:

**১. Database Tuning:**

PostgreSQL configuration আপডেট:

```sql
# /etc/postgresql/14/main/postgresql.conf

shared_buffers = 2GB
effective_cache_size = 6GB
work_mem = 64MB
maintenance_work_mem = 512MB
```

**২. Index তৈরি:**

যেসব ফিল্ডে প্রায়ই সার্চ করা হয় সেগুলোতে index:

```sql
CREATE INDEX idx_device_name ON dcim_device(name);
CREATE INDEX idx_ipaddress_address ON ipam_ipaddress(host(address));
```

**৩. Pagination বাড়ানো:**

User Preferences-এ:
```
Items per page: 100 (ডিফল্ট 25 থেকে বাড়ানো)
```

**৪. Saved Filters ইউজ:**

বারবার একই complex query না চালিয়ে Saved Filters ব্যবহার।

### Response Time Monitoring

একটা সিম্পল স্ক্রিপ্ট যা Nautobot API response time মাপে:

```python
import time
from pynautobot import api

nb = api(url="https://nautobot.skynet.bd", token="token")

start = time.time()
devices = nb.dcim.devices.all()
end = time.time()

print(f"Fetched {len(devices)} devices in {end-start:.2f} seconds")

# Target: <2 seconds for 120 devices
if (end - start) > 2:
    print("WARNING: Slow response time!")
```

প্রতি সপ্তাহে এই স্ক্রিপ্ট চালান। যদি রেসপন্স টাইম বাড়তে থাকে, optimization দরকার।

### সপ্তম চ্যালেঞ্জ: Data Quality Control

১২০ ডিভাইস মানে ১২০ জায়গায় ভুল হওয়ার সম্ভাবনা। Data Quality নিশ্চিত করা জরুরি।

### Weekly Data Audit

SkyNet প্রতি সপ্তাহে একটা ডেটা অডিট করে:

**সোমবার: Device Audit**
```
চেক করুন:
- সব Active ডিভাইসের Serial Number আছে কিনা
- সব ডিভাইসের Location সঠিক কিনা
- নেমিং কনভেনশন মানা হচ্ছে কিনা
```

**বুধবার: IP Audit**
```
চেক করুন:
- কোনো ডুপ্লিকেট IP নেই তো
- কোনো IP ভুল prefix-এ নেই তো
- DNS names unique কিনা
```

**শুক্রবার: Cable Audit**
```
চেক করুন:
- Critical cables ডকুমেন্টেড কিনা
- Cable status সঠিক কিনা (Connected/Planned)
```

### Automated Validation Script

একটা Python স্ক্রিপ্ট যা automatically validate করে:

```python
from pynautobot import api
from collections import Counter

nb = api(url="https://nautobot.skynet.bd", token="token")

print("=== SkyNet Data Quality Report ===\n")

# 1. Devices without serial numbers
devices = nb.dcim.devices.filter(status="active")
no_serial = [d for d in devices if not d.serial]
print(f"Devices without serial: {len(no_serial)}")
if no_serial:
    for d in no_serial[:5]:  # Show first 5
        print(f"  - {d.name}")

# 2. Duplicate IP addresses
ips = nb.ipam.ip_addresses.all()
ip_list = [ip.address for ip in ips]
duplicates = [ip for ip, count in Counter(ip_list).items() if count > 1]
print(f"\nDuplicate IPs: {len(duplicates)}")

# 3. Devices without location
no_location = [d for d in devices if not d.location]
print(f"\nDevices without location: {len(no_location)}")

# 4. Naming convention check
bad_names = []
for d in devices:
    # Should be: [Type]-DN-[Site]-[Role]-[Num]
    parts = d.name.split('-')
    if len(parts) != 5 or parts[1] != 'DN':
        bad_names.append(d.name)

print(f"\nDevices with non-standard names: {len(bad_names)}")
if bad_names:
    for name in bad_names[:5]:
        print(f"  - {name}")
```

এই স্ক্রিপ্ট প্রতি সোমবার সকালে cron দিয়ে চালান। রিপোর্ট ইমেইলে পাঠান।

### অষ্টম চ্যালেঞ্জ: Backup এবং Disaster Recovery

৫০ হাজার কাস্টমারের নেটওয়ার্ক ডাটা হারিয়ে গেলে বিপদ। Backup strategy আরো robust করা হয়েছে।

### Multi-Layer Backup

**Layer 1: Database Backup (দৈনিক)**
```bash
#!/bin/bash
# /home/skynet/backup-nautobot.sh

DATE=$(date +%Y%m%d_%H%M%S)
BACKUP_DIR="/backup/nautobot/database"

# PostgreSQL dump
docker exec nautobot-db-1 pg_dump -U nautobot nautobot > \
  $BACKUP_DIR/nautobot_$DATE.sql

# Compress
gzip $BACKUP_DIR/nautobot_$DATE.sql

# Keep last 30 days
find $BACKUP_DIR -name "*.sql.gz" -mtime +30 -delete

echo "Backup completed: $DATE"
```

**Layer 2: Full Export (সাপ্তাহিক)**
```python
# export-all-data.py
from pynautobot import api
import json
from datetime import datetime

nb = api(url="https://nautobot.skynet.bd", token="token")

data = {
    'export_date': datetime.now().isoformat(),
    'locations': [],
    'devices': [],
    'ip_addresses': [],
    'vlans': [],
}

# Export locations
for loc in nb.dcim.locations.all():
    data['locations'].append({
        'name': loc.name,
        'location_type': str(loc.location_type),
        'status': str(loc.status),
    })

# Export devices
for dev in nb.dcim.devices.all():
    data['devices'].append({
        'name': dev.name,
        'device_type': str(dev.device_type),
        'location': str(dev.location),
        'serial': dev.serial,
    })

# ... একইভাবে IP এবং VLAN

# Save to JSON
filename = f"nautobot_export_{datetime.now().strftime('%Y%m%d')}.json"
with open(filename, 'w') as f:
    json.dump(data, f, indent=2)

print(f"Export completed: {filename}")
```

**Layer 3: Offsite Backup (মাসিক)**

সব ব্যাকআপ Google Drive বা AWS S3-এ কপি করুন।

### Disaster Recovery Test

বছরে একবার DR test করুন:

১. একটা টেস্ট সার্ভারে Nautobot ইনস্টল করুন
২. সর্বশেষ ব্যাকআপ restore করুন
৩. যাচাই করুন সব ডেটা ঠিক আছে কিনা
৪. সময় মাপুন - কত দ্রুত restore করা যায়

SkyNet-এর টার্গেট: ৪ ঘন্টার মধ্যে ফুল restore।

### ৬ মাসের রোডম্যাপ - ৫ হাজার থেকে ৫০ হাজারে

এই স্কেলিং একদিনে হয়নি। ছয় মাস লেগেছে। এই ছিল রোডম্যাপ:

### মাস ১-২: Foundation Strengthening
- Location hierarchy তৈরি
- নেমিং কনভেনশন আপডেট
- পুরোনো ডিভাইস রিনেম
- Permission structure সেটআপ

### মাস ৩-৪: New Sites Onboarding
- উত্তরা পপ যুক্ত (প্রথম নতুন সাইট)
- বনানী পপ যুক্ত
- গুলশান পপ যুক্ত
- প্রতিটা সাইটে ফুল ডকুমেন্টেশন

### মাস ৫: IP এবং VLAN Restructuring
- নতুন IP blocks যুক্ত
- VLAN স্ট্রাকচার রিডিজাইন
- Migration প্ল্যানিং

### মাস ৬: Automation এবং Optimization
- Backup automation
- Data quality scripts
- Performance tuning
- Team training

### Key Metrics - আগে এবং পরে

কিছু সংখ্যা যা সফলতা প্রমাণ করে:

**২০২৪ (৫ হাজার কাস্টমার):**
- নতুন সাইট যুক্ত করতে সময়: ২ সপ্তাহ
- ডেটা এন্ট্রি ভুলের হার: ~১৫%
- NOC query response time: ৫-১০ মিনিট
- ম্যানুয়াল কাজ: ৭০%

**২০২৬ (৫০ হাজার কাস্টমার):**
- নতুন সাইট যুক্ত করতে সময়: ৩ দিন
- ডেটা এন্ট্রি ভুলের হার: ~৫%
- NOC query response time: ৩০ সেকেন্ড
- ম্যানুয়াল কাজ: ৩০%

### শিক্ষা এবং পরামর্শ

SkyNet-এর CTO জাহাঙ্গীর সাহেবের কিছু পরামর্শ:

**১. একবারে সব করতে যাবেন না**
"আমরা ভুল করেছিলাম প্রথমে সব আটটা সাইট একসাথে করতে গিয়ে। পরে বুঝলাম একটা একটা করে করলে ভালো হয়।"

**২. নেমিং কনভেনশন শুরুতেই ভবিষ্যৎ মাথায় রেখে করুন**
"আমাদের দুইবার নাম চেঞ্জ করতে হয়েছে। প্রথমবারই যদি স্কেলেবল কনভেনশন করতাম, এত ঝামেলা হতো না।"

**৩. Automation আগে থেকে শুরু করুন**
"যখন ছোট ছিলাম তখন মনে হয়েছিল ম্যানুয়ালি করি। বড় হয়ে গিয়ে automation করতে অনেক কষ্ট হয়েছে।"

**৪. টিমকে সাথে নিন**
"Nautobot শুধু একজনের জিনিস না। পুরো টিম যদি ইউজ না করে, তাহলে ডেটা আউটডেটেড হয়ে যায়।"

**৫. Data quality-তে কম্প্রোমাইজ করবেন না**
"একটা ভুল ডেটা এন্ট্রি পুরো নেটওয়ার্কে সমস্যা করতে পারে। কোয়ালিটি মেইনটেইন করুন।"

### এগিয়ে যাওয়ার পথ

SkyNet এখন ৫০ হাজার কাস্টমারে স্ট্যাবল। পরের লক্ষ্য ১ লক্ষ। সেজন্য তারা এখন প্রস্তুতি নিচ্ছে:

- **ঢাকা সাউথ সম্প্রসারণ:** নতুন zone যুক্ত করা
- **API-based provisioning:** নতুন কাস্টমার অটোমেটিক Nautobot-এ যুক্ত হবে
- **Monitoring integration:** Prometheus/Grafana সাথে Nautobot connect
- **Golden Config:** সব ডিভাইসের কনফিগ ব্যাকআপ এবং compliance check

পরের চ্যাপ্টারে আমরা দেখব একটা বাংলাদেশি আইএসপির সম্পূর্ণ রিয়েল কেস স্টাডি - কীভাবে তারা শুরু করেছিল, কী কী সমস্যার মুখোমুখি হয়েছিল, আর শেষপর্যন্ত কীভাবে সফল হয়েছিল।