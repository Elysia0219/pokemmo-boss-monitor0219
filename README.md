import requests
import re
import time

# ========== 你的配置（已填好） ==========
MIAO_CODE = "tHCu54O"
MONITOR_URL = "https://pokemmo.lanbizi.com/"
# ======================================

# 初始化上次内容变量
last_content = ""

def send_miao(msg):
    """发送妙推提醒，添加错误日志输出"""
    try:
        # 对消息内容做 URL 编码，避免特殊字符导致请求失败
        import urllib.parse
        encoded_msg = urllib.parse.quote(msg)
        url = f"https://miaotixing.com/trigger?id={MIAO_CODE}&text={encoded_msg}"
        headers = {
            "User-Agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/120.0.0.0 Safari/537.36"
        }
        # 添加响应状态码检查
        response = requests.get(url, headers=headers, timeout=10)
        response.raise_for_status()  # 非 200 状态码抛出异常
        print(f"妙推发送成功，响应：{response.text}")
    except Exception as e:
        # 不再静默失败，打印具体错误（关键！方便排查）
        print(f"妙推发送失败：{str(e)}")
        # 抛出异常让主流程捕获（可选，根据需要决定是否终止）
        raise

def extract_boss_info(html):
    """提取头目信息，增加容错处理"""
    info = {
        '名称': '未识别',
        '地点': '未识别',
        '时间': '未识别',
        '特性': '未识别',
        '蛋组': '未识别',
        '技能': '未识别'
    }
    # 正则匹配规则（优化匹配逻辑，减少失败概率）
    patterns = {
        '名称': r'(?:头目|名称)\s*[:：]\s*([^\n\r<>]+?)[\n\r<]',
        '地点': r'(?:地点|位置)\s*[:：]\s*([^\n\r<>]+?)[\n\r<]',
        '时间': r'(?:时间|起止)\s*[:：]\s*([^\n\r<>]+?)[\n\r<]',
        '特性': r'(?:特性|性格)\s*[:：]\s*([^\n\r<>]+?)[\n\r<]',
        '蛋组': r'(?:蛋组)\s*[:：]\s*([^\n\r<>]+?)[\n\r<]',
        '技能': r'(?:技能|配招)\s*[:：]\s*([^\n\r<>]+?)[\n\r<]'
    }
    
    for key, pattern in patterns.items():
        match = re.search(pattern, html, re.I | re.S)  # 增加 re.S 匹配换行
        if match:
            info[key] = match.group(1).strip()
    
    return info

def check():
    global last_content
    try:
        headers = {
            "User-Agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/120.0.0.0 Safari/537.36"
        }
        # 增加请求超时和状态码检查
        r = requests.get(MONITOR_URL, headers=headers, timeout=15)
        r.raise_for_status()  # 非 200 状态码直接抛出异常
        r.encoding = 'utf-8'
        html = r.text.strip()  # 去除首尾空白，避免微小差异导致误判

        # 核心修复：正确判断并更新 last_content
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
            # 关键修复：更新 last_content，避免重复推送
            last_content = html
            return msg
        else:
            print("暂无新头目或内容未变化")
            return "暂无新头目或内容未变化"
    except Exception as e:
        error_msg = f"检测失败：{str(e)}"
        print(error_msg)
        # 抛出异常让脚本返回非 0 码（方便 GitHub Actions 识别失败）
        raise Exception(error_msg)

if __name__ == "__main__":
    try:
        result = check()
        print(f"运行结果：{result}")
        # 正常退出返回 0
        exit(0)
    except Exception as e:
        print(f"脚本运行失败：{str(e)}")
        # 异常退出返回 1（替代 exit code 2，更规范）
        exit(1)
