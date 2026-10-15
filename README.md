# 两个人的30道菜 · 她的小灶台

一个鼓励女朋友做饭的小网站：记录「什么时候吃了什么菜」，给大厨（女朋友）付加工费、给食材费记账，支持打卡、菜谱、照片、账本统计。

## 功能

- **30 道菜单**：湘菜（香辣）8 道 + 南京菜（清淡）8 道 + 川菜 6 道 + 粤菜 5 道 + 家常菜 3 道，持续可加菜
- **每道菜**：成品摆盘主图 + 细节特写图（2 张）、食材清单、做法步骤、难度、加工费、预估食材费
- **打卡**：记录日期、好吃程度（1-5 星）、一句话点评；加工费 + 食材费自动入账
- **小账本**：加工费 / 食材费累计、钱包流水双线图、最受欢迎 TOP5、各菜系解锁进度
- **改价**：卡片上铅笔图标随时调整加工费 / 食材费，只影响以后打卡
- **备份**：JSON 导出 / 导入（localStorage 或云端同步）

## 技术栈

- 单文件 HTML + 原生 JS（无构建），ECharts 图表
- 图片为 AI 生成的家常菜摄影，静态存放于 `assets/dishes/`
- 数据持久化：localStorage（离线缓存）→ Supabase（云端为主，多设备同步）

## 本地打开

直接用浏览器打开 `两个人的30道菜.html` 即可（同目录 `assets/` 需保持完整）。

## 部署（GitHub Pages）

仓库已配置，启用后访问 `https://Danceiny.github.io/love-menu/`。

## Supabase 配置

1. 在 [supabase.com](https://supabase.com) 创建免费项目
2. 打开 **SQL Editor** 执行下方建表 SQL（含 RLS 策略 + 密码锁配置表）
3. 在 **Project Settings → API** 复制 Project URL 与 anon public key
4. 填入页面顶部的 `SUPABASE_URL` / `SUPABASE_ANON_KEY`（或页面「设置」中的云端同步配置）

### 建表 SQL

```sql
-- 菜单表
create table if not exists public.dishes (
  id bigint primary key,
  name text not null,
  cat text not null,
  price int not null default 0,
  ing int not null default 0,
  diff int not null default 1,
  note text default '',
  imgs text[] default '{}',
  recipe jsonb default '{"ing":[],"steps":[]}',
  custom boolean default false,
  updated_at timestamptz default now()
);

-- 打卡记录表
create table if not exists public.records (
  id bigint primary key,
  dish_id bigint not null,
  date text not null,
  rating int not null default 5,
  comment text default '',
  fee int not null default 0,
  ing_fee int not null default 0,
  created_at timestamptz default now()
);

-- 站点配置表（存密码锁等）
create table if not exists public.site_config (
  key text primary key,
  value text not null
);

-- RLS：允许匿名读写（本项目为情侣私有数据源，靠密码锁防误写）
alter table public.dishes enable row level security;
alter table public.records enable row level security;
alter table public.site_config enable row level security;

create policy "anon all dishes" on public.dishes for all to anon using (true) with check (true);
create policy "anon all records" on public.records for all to anon using (true) with check (true);
create policy "anon all config" on public.site_config for all to anon using (true) with check (true);

-- 密码锁：把密码写进配置（把 你的密码 换成实际密码，如 herkitchen2026）
insert into public.site_config (key, value) values ('access_pass', '你的密码')
  on conflict (key) do update set value = excluded.value;
```

> 注意：anon key 可直接操作数据，密码锁是「防君子不防黑客」的便利措施；介意的话可改用 Supabase Auth 登录体系（需要账号系统，暂未接入）。

## 目录

```
两个人的30道菜.html   单页应用（全部功能）
assets/hero.jpg      封面图
assets/dishes/       60 张菜品图（主图 + 细节）
_backup/             历史版本备份
```
