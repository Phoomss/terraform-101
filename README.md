# Terraform 101: AWS EC2 & VPC Demo

โปรเจกต์นี้เป็นตัวอย่างการเริ่มต้นใช้งาน Terraform (Terraform 101) เพื่อสร้างโครงสร้างพื้นฐาน (Infrastructure) บน AWS ประกอบด้วย VPC, Subnet, Security Group และ EC2 Instance (Web Server) อย่างง่าย ในภูมิภาค **Singapore (ap-southeast-1)**

---

## 🛠 สิ่งที่ต้องเตรียมก่อนเริ่มใช้งาน (Prerequisites)

1. **ติดตั้ง Terraform** (เวอร์ชัน >= 1.5)
   - **macOS (via Homebrew):**
     ```bash
     brew tap hashicorp/tap
     brew install hashicorp/tap/terraform
     ```
   - **Windows (via Chocolatey):**
     ```powershell
     choco install terraform
     ```
   - หรือดาวน์โหลดโดยตรงจาก [Terraform Downloads](https://developer.hashicorp.com/terraform/downloads)

2. **ติดตั้ง AWS CLI และตั้งค่า Credentials**
   - ติดตั้ง AWS CLI และทำการ Login ด้วยคีย์ของคุณ:
     ```bash
     aws configure
     ```
     กรอก `AWS Access Key ID`, `AWS Secret Access Key`, และ Default region (แนะนำ: `ap-southeast-1`)

3. **เตรียมคู่กุญแจ SSH Key Pair**
   - โปรเจกต์นี้ตั้งค่าให้ใช้ Key Pair ชื่อ `terraform-key` สำหรับเชื่อมต่อกับ EC2
   - คุณสามารถสร้างผ่าน AWS Console หรือใช้คำสั่ง AWS CLI ดังนี้:
     ```bash
     aws ec2 create-key-pair --key-name terraform-key --query 'KeyMaterial' --output text > terraform-key.pem
     chmod 400 terraform-key.pem
     ```
   - *หมายเหตุ: ไฟล์ `terraform-key.pem` จะถูกปฏิเสธไม่ให้บันทึกลง Git โดยอัตโนมัติผ่านระบบ `.gitignore` เพื่อความปลอดภัย*

---

## 📂 โครงสร้างโปรเจกต์ (Project Structure)

- [main.tf](file:///Users/mac/Desktop/workspace/terraform-demo/main.tf) - ไฟล์หลักที่ใช้ประกาศทรัพยากร (VPC, Subnet, Route Table, Security Group, EC2 Instance)
- [variables.tf](file:///Users/mac/Desktop/workspace/terraform-demo/variables.tf) - ไฟล์กำหนดตัวแปร (Variables) เช่น CIDR Block, AWS Region, Instance Type
- [outputs.tf](file:///Users/mac/Desktop/workspace/terraform-demo/outputs.tf) - ไฟล์ระบุค่าผลลัพธ์ที่ต้องการแสดงหลังจาก Deploy สำเร็จ (เช่น Public IP ของ Instance)
- [.gitignore](file:///Users/mac/Desktop/workspace/terraform-demo/.gitignore) - ไฟล์ตั้งค่าไม่ให้ Git ติดตามโฟลเดอร์ชั่วคราว คีย์ลับ หรือไฟล์ State ของ Terraform

---

## 🚀 ขั้นตอนการรันและการทำงาน (How to Run)

เปิด Terminal ในโฟลเดอร์โปรเจกต์นี้แล้วทำตามขั้นตอนดังต่อไปนี้:

### 1. เริ่มต้นโปรเจกต์ (Initialize)
ดาวน์โหลดและเตรียม Provider plugins ที่จำเป็น (ในที่นี้คือ AWS Provider)
```bash
terraform init
```

### 2. ตรวจสอบแผนการสร้าง (Plan)
สร้างรายงานตรวจสอบว่า Terraform จะสร้าง, แก้ไข หรือลบทรัพยากรอะไรบ้างบน AWS Cloud
```bash
terraform plan
```

### 3. เริ่มการติดตั้งจริง (Apply)
สั่งให้สร้างทรัพยากรบน AWS ตามแผน (พิมพ์ `yes` เพื่อยืนยันเมื่อหน้าจอสอบถาม)
```bash
terraform apply
```
*หลังจากรันคำสั่งสำเร็จ ระบบจะแสดงผลลัพธ์ `instance_public_ip` และ `instance_id` ขึ้นมาให้ใช้งาน*

### 4. การทำลายทรัพยากรเมื่อเลิกใช้งาน (Destroy)
ลบทรัพยากรทั้งหมดที่โปรเจกต์นี้สร้างขึ้นเพื่อหลีกเลี่ยงการเสียค่าใช้จ่ายสะสมบน AWS
```bash
terraform destroy
```

---

## ⚙️ ตัวแปรและผลลัพธ์ (Variables & Outputs)

### ตัวแปร (Variables)
คุณสามารถปรับแต่งค่าเหล่านี้ได้ในไฟล์ [variables.tf](file:///Users/mac/Desktop/workspace/terraform-demo/variables.tf) หรือระบุผ่านไฟล์ `.tfvars`

| ชื่อตัวแปร | รายละเอียด | ค่าเริ่มต้น (Default) |
| :--- | :--- | :--- |
| `aws_region` | AWS Region ที่ต้องการ Deploy | `ap-southeast-1` (Singapore) |
| `vpc_cidr` | Network IP Range สำหรับ VPC | `10.0.0.0/16` |
| `public_subnet_cidr` | Network IP Range สำหรับ Subnet | `10.0.1.0/24` |
| `instance_type` | ขนาดสเปกของเครื่อง EC2 | `t3.micro` |

### ผลลัพธ์ (Outputs)
แสดงหลังจากรันคำสั่ง `terraform apply` สำเร็จ:

- `instance_public_ip` - เลข Public IP ของ EC2 Instance ที่สร้างขึ้น
- `instance_id` - ID ของ EC2 Instance สำหรับใช้อ้างอิงในระบบ AWS

---

## 🔒 ความปลอดภัยและข้อควรระวัง (Security & Best Practices)

- **ห้าม Commit ไฟล์ state และกุญแจส่วนตัวขึ้น Git**: ไฟล์ `.terraform.tfstate`, `terraform.tfvars` (หากมีข้อมูลสำคัญ), และคีย์ `*.pem` จะมีข้อมูลที่ละเอียดอ่อนสูง เช่น รหัสผ่าน หรือสเปกโครงสร้างที่อาจเป็นช่องโหว่ความปลอดภัย โดยไฟล์เหล่านี้ได้รับการบล็อกผ่าน [.gitignore](file:///Users/mac/Desktop/workspace/terraform-demo/.gitignore) เรียบร้อยแล้ว
- **สเปกของ AMI**: ใน `main.tf` มีการล็อกไอดี Ubuntu AMI (`ami-0ba6f2c4de657798c`) เฉพาะสำหรับ Region `ap-southeast-1` เท่านั้น หากเปลี่ยน Region จะต้องระบุ AMI ID ของ Region นั้นๆ ด้วย
