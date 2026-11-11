import os
from Crypto.Cipher import AES
from Crypto.Util import Counter

# القيم الثابتة المستخدمة في تشفير معرض شاومي (MIUI Gallery)
SECRET_KEY = bytes.fromhex('30313233343536373839616263646566')  # المفتاح الافتراضي لمعظم أجهزة شاومي
AES_IV = bytes([17, 19, 33, 35, 49, 51, 65, 67, 81, 83, 97, 102, 103, 104, 113, 114])

def decrypt_lsav_to_mp4(input_file, output_file):
    # تهيئة عداد AES CTR
    iv_int = int.from_bytes(AES_IV, byteorder='big')
    ctr = Counter.new(128, initial_value=iv_int)
    cipher = AES.new(SECRET_KEY, AES.MODE_CTR, counter=ctr)

    with open(input_file, 'rb') as f_in:
        data = bytearray(f_in.read())

    # شاومي تشفر أول 1024 بايت فقط (Header) في ملفات الفيديو .lsav
    header_size = min(1024, len(data))
    decrypted_header = cipher.decrypt(bytes(data[:header_size]))
    
    # استبدال الترويسة المشفرة بالمفكوكة
    data[:header_size] = decrypted_header

    with open(output_file, 'wb') as f_out:
        f_out.write(data)
    
    print(f"تم التحويل بنجاح! الفيديو محفوظ باسم: {output_file}")

# --- طريقة الاستخدام ---
# ضع اسم ملفك هنا
input_filename = "video.lsav" 
output_filename = "recovered_video.mp4"

if os.path.exists(input_filename):
    decrypt_lsav_to_mp4(input_filename, output_filename)
else:
    print("الملف غير موجود، تأكد من كتابة الاسم والمسار بشكل صحيح.")
