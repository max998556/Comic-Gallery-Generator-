#【使用方式】
1. 準備母資料夾，內部放置多個子資料夾
2. 每個子資料夾代表一部漫畫（章節）
3. 子資料夾內放入圖片檔案（jpg、png 等）
4. 執行此程式，輸入母資料夾路徑
5. 自動生成 HTML 網頁和目錄

【資料夾結構範例】
MyComics/
├── 001_進擊的巨人_第1卷/
│   ├── 00001.jpg
│   ├── 00002.jpg
│   └── ...
├── 002_進擊的巨人_第2卷/
│   ├── 00001.jpg
│   └── ...
└── ...

【功能】
- 自動生成漫畫閱讀網頁
- 無限滾動載入章節
- 支援圖片寬度調整
- 懸浮工具列快速操作
- 黑色背景護眼設計

【輸入】
輸入母資料夾完整路徑，例如：
D:\\MyComics 或 /home/user/MyComics

【輸出】
生成 HTML 檔案可直接用瀏覽器開啟
    """)

def main():
    show_help()  # 程式啟動時顯示說明
    while True:


    ------------------以下是腳本.py----------------------------------

    import os
import math
import json

# 常數
EXTS = ('.jpg', '.jpeg', '.png', '.gif', '.bmp', '.webp', '.tiff', '.tif', '.ico', '.svg', '.heic', '.heif', '.raw', '.psd', '.ai')
WIDTHS = ['auto', 'calc(100vw - 120px)', 'calc(100vw - 80px)', 'calc(100vw - 10px)']

# 漫畫頁面 HTML
COMIC_HTML = '''<!DOCTYPE html>
<html lang="zh-TW">
<head><meta charset="UTF-8"><title>{cn}</title><style>
body{{margin:0;padding:0;background:#000;color:#666;font-family:Arial;text-align:center}}
.container{{display:flex;flex-direction:column;align-items:center}}
.chapter-title{{color:#0f0;margin:20px 0;font-size:20px;padding:10px;background:#111}}
img{{display:block;width:auto;height:auto;margin:0;padding:0;border:none;object-fit:contain}}
.loading{{color:#ccc;padding:20px;font-size:16px}}
.floating-toolbar{{position:fixed;bottom:20px;right:20px;width:35px;height:35px;background:rgba(0,0,0,0.3);border:1px solid #555;border-radius:50%;cursor:move;z-index:1000;transition:all 0.1s ease;user-select:none;display:flex;align-items:center;justify-content:center;padding:0}}
.toolbar-content{{display:none;flex-direction:column;gap:1px;align-items:center;opacity:1;transition:opacity 0.1s ease}}
.floating-toolbar:hover .toolbar-content{{display:flex;opacity:1}}
.floating-toolbar .toolbar-content{{opacity:0;pointer-events:none}}
.toolbar-button{{padding:0 24px;background:transparent;color:#ccc;border:none;border-radius:1px;cursor:pointer;font-size:25px;text-decoration:none;text-align:center;white-space:nowrap}}
.toolbar-button:hover{{background:rgba(255,255,255,0.1)}}
*{{color:#666!important}}
</style></head>
<body>
<div class="container" id="c"></div>
<div id="l" class="loading" style="display:none">⏳ 正在載入下一章...</div>
<div class="floating-toolbar" id="t" draggable="true">
<div class="toolbar-content" id="tc">
<button class="toolbar-button" onclick="lp()" title="上一章">🔼</button>
<button class="toolbar-button" onclick="rc()" title="章節刷新">🔃</button>
<button class="toolbar-button" onclick="tw()" title="調整寬度">↔️</button>
<button class="toolbar-button" onclick="ln()" title="下一章">🔽</button>
<a href="../{pn}.html" class="toolbar-button" title="返回目錄">📚</a>
</div>
</div>
<script>
// 配置cfg 狀態st
const cfg={{"ac":{ac},"ci":{ci},"wm":{wm}}};
const st={{"lc":[],"il":false,"id":false,"wm":0,"ht":null,"do":{{"x":0,"y":0}}}};
// DOM
const dm={{"c":document.getElementById('c'),"l":document.getElementById('l'),"t":document.getElementById('t'),"tc":document.getElementById('tc')}};

// 初始化章節
function ic(i){{
if(st.lc.includes(i)||i<0||i>=cfg.ac.length)return;
const cm=cfg.ac[i];
const cd=document.createElement('div');
cd.id='c'+i;
let h='<h2 class="chapter-title">'+cm.n+'</h2>';
cm.i.forEach(img=>{{h+='<img src="../'+cm.p+'/'+img+'" style="width:'+cfg.wm[st.wm]+'">';}});
cd.innerHTML=h;
dm.c.appendChild(cd);
st.lc.push(i);
// 只保留3章，刪除最舊的
if(st.lc.length>3){{
const oi=st.lc.shift();
const od=document.getElementById('c'+oi);
if(od)od.remove();
}}
}}

// 刷新章節
function rc(){{
dm.c.innerHTML='';st.lc=[];ic(cfg.ci);window.scrollTo(0,0);
}}

// 切換寬度
function tw(){{
st.wm=(st.wm+1)%cfg.wm.length;
document.querySelectorAll('img').forEach(img=>{{img.style.width=cfg.wm[st.wm];}});
}}

// 上一章
function lp(){{
if(cfg.ci<=0)return;
cfg.ci--;
if(!st.lc.includes(cfg.ci))ic(cfg.ci);
window.scrollTo(0,0);
}}

// 下一章
function ln(){{
if(st.il||cfg.ci>=cfg.ac.length-1)return;
st.il=true;dm.l.style.display='block';
setTimeout(()=>{{
cfg.ci++;
ic(cfg.ci);
st.il=false;dm.l.style.display='none';
}},500);
}}

// 滾動到底部
window.addEventListener('scroll',()=>{{
if((window.innerHeight+window.scrollY)>=document.body.offsetHeight-100){{
if(!st.il&&cfg.ci<cfg.ac.length-1){{
st.il=true;dm.l.style.display='block';
setTimeout(()=>{{
cfg.ci++;
ic(cfg.ci);
st.il=false;dm.l.style.display='none';
}},500);
}}
}}
}});

// 拖動
dm.t.addEventListener('dragstart',e=>{{st.id=true;st.do.x=e.clientX-dm.t.offsetLeft;st.do.y=e.clientY-dm.t.offsetTop;}});
document.addEventListener('dragover',e=>e.preventDefault());
document.addEventListener('drop',e=>{{
if(st.id){{
dm.t.style.left=(e.clientX-st.do.x)+'px';
dm.t.style.top=(e.clientY-st.do.y)+'px';
dm.t.style.right='auto';
dm.t.style.bottom='auto';
st.id=false;
}}
}});

// 懸停
dm.t.addEventListener('mouseenter',()=>{{clearTimeout(st.ht);dm.tc.style.opacity='1';dm.tc.style.pointerEvents='auto';}});
dm.t.addEventListener('mouseleave',()=>{{st.ht=setTimeout(()=>{{dm.tc.style.opacity='0';dm.tc.style.pointerEvents='none';}},800);}});

// 初始化
ic(cfg.ci);
</script>
</body>
</html>'''

# 目錄頁面 HTML
INDEX_HTML = '''<!DOCTYPE html>
<html lang="zh-TW">
<head><meta charset="UTF-8"><title>{pn}</title><style>
body{{margin:0;padding:0;background:#000;color:#666;font-family:Arial;text-align:center}}
h1{{color:#ccc;margin:30px 0;font-size:32px}}
.comic-table{{margin:20px auto;border-collapse:collapse;background:#111}}
.comic-table td{{padding:15px;border:1px solid #333;text-align:center}}
.comic-table a{{color:#ccc;text-decoration:none;display:block;padding:10px}}
.comic-table a:hover{{background:#333;border-radius:5px}}
</style></head>
<body>
<h1>📚 {pn}</h1>
{th}
</body>
</html>'''

# 取得圖片gi 產生漫畫頁面gch 產生目錄gih
def gi(fp):
    return sorted([f for f in os.listdir(fp) if f.lower().endswith(EXTS)])

def gch(fp, cn, img, ac, ci, pf):
    try:
        pn = os.path.basename(pf)
        ac_json = json.dumps([{"n": c["name"], "p": c["path"], "i": c["images"]} for c in ac], ensure_ascii=False)
        wm_json = json.dumps(WIDTHS, ensure_ascii=False)
        
        html = COMIC_HTML.format(cn=cn, pn=pn, ac=ac_json, ci=ci, wm=wm_json)
        
        op = os.path.join(fp, '_0.html')
        with open(op, 'w', encoding='utf-8') as f:
            f.write(html)
#        print(f"✅ {cn} 完成 → {op}，共 {len(img)} 張")
        return True
    except Exception as e:
        print(f"❌ {cn} 錯誤：{e}")
        return False

def gih(pf, pn, cm):
    try:
        cols = 5
        rows = math.ceil(len(cm) / cols)
        
        th = '<table class="comic-table">\n'
        for row in range(rows):
            th += '<tr>\n'
            for col in range(cols):
                idx = row * cols + col
                if idx < len(cm):
                    th += f'<td><a href="{cm[idx] ["path"]}/_0.html">{cm[idx] ["name"]}</a></td>\n'
                else:
                    th += '<td></td>\n'
            th += '</tr>\n'
        th += '</table>'
        
        html = INDEX_HTML.format(pn=pn, th=th)
        
        op = os.path.join(pf, f"{pn}.html")
        with open(op, 'w', encoding='utf-8') as f:
            f.write(html)
        print(f"✅ 目錄   {op}，共 {len(cm)} 本")
        return True
    except Exception as e:
        print(f"❌ 目錄錯誤：{e}")
        return False

# 掃描
def sc(pf):
    cm = []
    for item in os.listdir(pf):
        ip = os.path.join(pf, item)
        if os.path.isdir(ip):
            img = gi(ip)
            if img:
                cm.append({"name": item, "path": item, "full_path": ip, "images": img})
    return cm

# 主程式
def main():
    while True:
        try:
            print("\n請輸入母資料夾路徑（輸入 'q' 退出）：")
            pf = input().strip()

            if pf.lower() == 'q':
                print("👋 再見！")
                break

            if not os.path.isdir(pf):
                print(f"❌ 路徑不存在：{pf}")
                continue

            pn = os.path.basename(pf)
            print(f"📁 母資料夾：{pn}")

            cm = sc(pf)

            if not cm:
                print("❌ 沒有找到漫畫資料夾")
                continue

            sc_cnt = sum(1 for idx, c in enumerate(cm) if gch(c["full_path"], c["name"], c["images"], cm, idx, pf))

            if gih(pf, pn, cm):
                print(f"🎉 完成！成功 {sc_cnt}/{len(cm)} 本")

        except Exception as e:
            print(f"❌ 錯誤：{e}")

if __name__ == "__main__":
    try:
        main()
    finally:
        print("\n按 Enter 結束...")
        input()
