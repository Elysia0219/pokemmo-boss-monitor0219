import requests
import re
import time

# ========== 你的配置（已填好） ==========
MIAO_CODE = "tHCu54O"
MONITOR_URL = "https://pokemmo.lanbizi.com/"
# ======================================

last_content = ""

def send_miao(msg):
    try:
        url = f"https://miaotixing.com/trigger?id={MIAO_CODE}&text={msg}"
        headers = {
            "User-Agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/120.0.0.0 Safari/537.36"
        }
        requests.get(url, headers=headers, timeout=10)
    except Exception:
        pass

def extract_boss_info(html):
    info = {}
    name_match = re.search(r'(?:头目|名称)\s*[:：]\s*([^\n\r<>]+)', html, re.I)
    place_match = re.search(r'(?:地点|位置)\s*[:：]\s*([^\n\r<>]+)', html, re.I)
    time_match = re.search(r'(?:时间|起止)\s*[:：]\s*([^\n\r<>]+)', html, re.I)
    ability_match = re.search(r'(?:特性|性格)\s*[:：]\s*([^\n\r<>]+)', html, re.I)
    egg_match = re.search(r'(?:蛋组)\s*[:：]\s*([^\n\r<>]+)', html, re.I)
    skill_match = re.search(r'(?:技能|配招)\s*[:：]\s*([^\n\r<>]+)', html, re.I)

    info['名称'] = name_match.group(1).strip() if name_match else "未识别"
    info['地点'] = place_match.group(1).strip() if place_match else "未识别"
    info['时间'] = time_match.group(1).strip() if time_match else "未识别"
    info['特性'] = ability_match.group(1).strip() if ability_match else "未识别"
    info['蛋组'] = egg_match.group(1).strip() if egg_match else "未识别"
    info['技能'] = skill_match.group(1).strip() if skill_match else "未识别"
    return info

def check():
    global last_content
    try:
        headers = {
            "User-Agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/120.0.0.0 Safari/537.36"
        }
        r = requests.get(MONITOR_URL, headers=headers, timeout=15)
        r.encoding = 'utf-8'
        html = r.text

        if ("头目" in html or "出现" in html) and html != last_content:
            info = extract_boss_info(html)
            msg = (
                f"【Pokemmo头目出现】\n"
                f"名称：{info['名称']}\n"
                f"地点：{info['地点']}\n"
                f"时间：{info['时间']}\n"
                f"特性：{info['特性']}\n"
                f"蛋组：{info['蛋组']}\n"
                f"技能：{info['技能']}"
            )
            print("检测到新头目，发送提醒：")
            print(msg)
            send_miao(msg)
            return msg
        else:
            print("暂无新头目")
            return "暂无新头目"
    except Exception as e:
        print(f"检测失败：{e}")
        return f"检测失败：{e}"

if __name__ == "__main__":
    result = check()
    print(result)
