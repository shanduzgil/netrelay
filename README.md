NetRelay

NetRelay یک کتابخانهٔ پایتونی برای مدیریت هوشمند رله‌های HTTP است؛ ابزاری برای ساخت ارتباطی پایدارتر با APIها و سرویس‌های خارجی، با پشتیبانی از Health Check، Failover خودکار، Circuit Breaker و Rate Limiter.

با NetRelay می‌توانید چندین endpoint را به‌عنوان رله تعریف کنید، سلامت آن‌ها را به‌صورت مداوم بررسی کنید و در هر درخواست، بهترین مسیر در دسترس را انتخاب کنید. اگر یک مسیر از کار بیفتد، NetRelay به‌صورت خودکار مسیر بعدی را امتحان می‌کند تا ارتباط شما پایدار و قابل‌اعتماد بماند.

---

فهرست مطالب

- "ویژگی‌ها" (#ویژگی‌ها)
- "نصب" (#نصب)
- "شروع سریع" (#شروع-سریع)
- "پیکربندی" (#پیکربندی)
- "استفاده در پایتون" (#استفاده-در-پایتون)
- "CLI" (#cli)
- "نمونه‌های کاربردی" (#نمونه‌های-کاربردی)
- "API Reference" (#api-reference)
- "توسعه و مشارکت" (#توسعه-و-مشارکت)
- "مجوز" (#مجوز)

---

ویژگی‌ها

- مدیریت چندین Relay به‌صورت هم‌زمان
- Health Check مداوم برای بررسی در دسترس بودن و تأخیر
- انتخاب هوشمند بهترین رله بر اساس چند استراتژی:
  - "latency"
  - "priority"
  - "weighted"
  - "random"
- Failover خودکار در صورت شکست یک مسیر
- Circuit Breaker برای خارج کردن موقت رله‌های ناسالم از مدار
- Rate Limiter برای کنترل تعداد درخواست‌ها
- پشتیبانی از پیکربندی JSON
- CLI برای مشاهده وضعیت، بررسی سلامت و ارسال درخواست
- API آسنکرون با "aiohttp"
- مناسب برای انتشار و استفاده از طریق PyPI

---

نصب

برای نصب آخرین نسخه:

pip install netrelay

برای نصب همراه با وابستگی‌های توسعه:

pip install netrelay[dev]

نیازمندی: Python 3.10 یا بالاتر

---

شروع سریع

1) فایل پیکربندی بسازید

فایل "relays.json":

{
  "relays": [
    {
      "name": "api-primary",
      "url": "https://api.example.com",
      "weight": 100,
      "priority": 1,
      "timeout": 5,
      "probe_path": "/health",
      "healthy_threshold": 2,
      "unhealthy_threshold": 2
    },
    {
      "name": "api-secondary",
      "url": "https://api2.example.com",
      "weight": 50,
      "priority": 2,
      "timeout": 5,
      "probe_path": "/health"
    }
  ],
  "selection": {
    "strategy": "latency",
    "retry_on_failure": true,
    "max_failover_attempts": 2
  },
  "health": {
    "interval": 30,
    "method": "GET",
    "probe_path": "/health",
    "timeout": 5
  }
}

2) در پایتون استفاده کنید

import asyncio
from netrelay import RelayManager

async def main():
    manager = RelayManager("relays.json")
    await manager.start()

    status, body, headers, relay_name = await manager.request("GET", "/data")
    print(f"Response from {relay_name}: {status}")
    print(body)

    await manager.stop()

asyncio.run(main())

3) یا از CLI استفاده کنید

netrelay list --config relays.json
netrelay health --config relays.json
netrelay request GET /data --config relays.json

---

پیکربندی

NetRelay از طریق فایل JSON پیکربندی می‌شود. بیشتر تنظیمات مقدار پیش‌فرض دارند، بنابراین می‌توانید فقط بخش‌های موردنیاز را مشخص کنید.

ساختار کلی پیکربندی

- "relays" — فهرست رله‌ها
- "selection" — تنظیمات انتخاب رله
- "health" — تنظیمات بررسی سلامت
- "rate_limit" — محدودیت نرخ
- "circuit_breaker" — تنظیمات مدارشکن
- "session" — تنظیمات نشست HTTP

---

تنظیمات هر Relay

فیلد| نوع| مقدار پیش‌فرض| توضیح
"name"| "string"| الزامی| نام یکتای رله
"url"| "string"| الزامی| آدرس پایهٔ رله
"weight"| "int"| "100"| وزن در استراتژی weighted
"priority"| "int"| "1"| اولویت؛ عدد کمتر یعنی اولویت بالاتر
"timeout"| "float"| "5.0"| زمان انتظار درخواست‌ها (ثانیه)
"probe_path"| "string"| "/get"| مسیر probe برای health check
"healthy_threshold"| "int"| "1"| تعداد موفقیت متوالی برای سالم شدن
"unhealthy_threshold"| "int"| "2"| تعداد خطای متوالی برای ناسالم شدن

---

تنظیمات انتخاب ("selection")

فیلد| نوع| مقدار پیش‌فرض| توضیح
"strategy"| "string"| ""latency""| استراتژی انتخاب: "latency", "priority", "weighted", "random"
"retry_on_failure"| "bool"| "true"| در صورت شکست، رلهٔ دیگری امتحان شود یا نه
"max_failover_attempts"| "int"| "3"| حداکثر تعداد تلاش برای failover

---

تنظیمات سلامت ("health")

فیلد| نوع| مقدار پیش‌فرض| توضیح
"interval"| "float"| "30.0"| فاصلهٔ بین بررسی‌های سلامت (ثانیه)
"method"| "string"| ""GET""| متد HTTP برای probe
"probe_path"| "string"| ""/get""| مسیر probe
"timeout"| "float"| "5.0"| زمان انتظار برای probe
"success_status_min"| "int"| "200"| کمترین کد وضعیت موفق
"success_status_max"| "int"| "399"| بیشترین کد وضعیت موفق

---

تنظیمات محدودیت نرخ ("rate_limit")

فیلد| نوع| مقدار پیش‌فرض| توضیح
"enabled"| "bool"| "false"| فعال بودن Rate Limiter
"requests_per_second"| "float"| "50"| تعداد درخواست مجاز در ثانیه
"burst"| "int"| "100"| حداکثر تعداد درخواست ناگهانی

---

تنظیمات مدارشکن ("circuit_breaker")

فیلد| نوع| مقدار پیش‌فرض| توضیح
"failure_threshold"| "int"| "3"| تعداد خطاهای متوالی برای باز شدن مدار
"recovery_timeout"| "float"| "30.0"| زمان انتظار تا تلاش دوباره
"half_open_max_requests"| "int"| "1"| تعداد درخواست آزمایشی در حالت نیمه‌باز

---

تنظیمات نشست ("session")

فیلد| نوع| مقدار پیش‌فرض| توضیح
"timeout"| "float"| "10.0"| زمان انتظار کلی هر درخواست
"trust_env"| "bool"| "true"| استفاده از متغیرهای محیطی برای proxy
"headers"| "object"| "{"User-Agent": "NetRelay/1.1.0", "Accept": "*/*"}"| هدرهای پیش‌فرض

---

استفاده در پایتون

ایجاد و راه‌اندازی Manager

from netrelay import RelayManager

manager = RelayManager("config.json")
await manager.start()

بررسی سلامت به‌صورت دستی

await manager.refresh_health()

for relay in manager.relays:
    print(f"{relay.name}: healthy={relay.healthy}, latency={relay.last_check.latency_ms} ms")

ارسال درخواست

status, body, headers, relay_name = await manager.request("GET", "/users")
print(f"Status: {status}, from: {relay_name}")

ارسال داده و هدر سفارشی

status, body, headers, relay_name = await manager.request(
    "POST",
    "/login",
    data={"username": "admin", "password": "123"},
    headers={"X-Custom": "value"}
)

مشاهدهٔ خلاصهٔ وضعیت

summary = manager.summary()
print(summary)
# {'total': 2, 'healthy': 1, 'best': 'api-primary', 'strategy': 'latency'}

متوقف کردن

await manager.stop()

---

CLI

پس از نصب، دستور "netrelay" در دسترس خواهد بود.

نمایش وضعیت رله‌ها

netrelay list --config config.json

بررسی سلامت

netrelay health --config config.json

ارسال درخواست HTTP

netrelay request GET /api/data --config config.json

پایش مداوم

netrelay watch --config config.json --interval 10

---

نمونه‌های کاربردی

1) بالانس بار بین چند API

from netrelay import RelayManager

manager = RelayManager()
manager.config["relays"] = [
    {"name": "node1", "url": "http://node1.local:8000"},
    {"name": "node2", "url": "http://node2.local:8000"},
    {"name": "node3", "url": "http://node3.local:8000"},
]
manager.config["selection"]["strategy"] = "random"

await manager.start()

2) استفاده از چند endpoint واسط

{
  "relays": [
    {"name": "proxy-us", "url": "http://proxy-us:8080"},
    {"name": "proxy-eu", "url": "http://proxy-eu:8080"},
    {"name": "proxy-tor", "url": "socks5://127.0.0.1:9050"}
  ],
  "selection": {
    "strategy": "latency",
    "max_failover_attempts": 5
  },
  "circuit_breaker": {
    "failure_threshold": 2,
    "recovery_timeout": 60
  }
}

«NetRelay خودش پروکسی نیست؛ بلکه این امکان را می‌دهد که endpointهای واقعی را به‌عنوان رله تعریف و مدیریت کنید.»

3) محافظت از کلاینت در برابر قطعی API

import asyncio
from netrelay import RelayManager

manager = RelayManager("config.json")
await manager.start()

for i in range(100):
    try:
        status, body, headers, name = await manager.request("GET", f"/items/{i}")
        process(body)
    except RuntimeError:
        print("All relays failed. Waiting...")
        await asyncio.sleep(5)

---

API Reference

"RelayManager"

class RelayManager:
    def __init__(self, config_path: Optional[str] = None)
    async def start() -> None
    async def stop() -> None
    async def refresh_health() -> List[HealthResult]
    async def request(method: str, path_or_url: str, **kwargs) -> Tuple[int, str, Dict[str, str], str]
    async def get_session() -> aiohttp.ClientSession
    def choose_relay() -> Optional[Relay]
    def summary() -> Dict[str, Any]
    def status_rows() -> List[Dict[str, Any]]
    def reload() -> None

توضیحات:

- "request" از "data", "json" و "headers" پشتیبانی می‌کند.
- "summary" خلاصه‌ای از تعداد کل، تعداد سالم‌ها، مدارهای باز، بهترین رله و استراتژی انتخاب را برمی‌گرداند.
- "status_rows" اطلاعات هر رله را برای نمایش در CLI یا UI فراهم می‌کند.

---

"Relay"

@dataclass
class Relay:
    name: str
    url: str
    weight: int
    priority: int
    timeout: float
    probe_path: str
    healthy_threshold: int
    unhealthy_threshold: int
    healthy: bool
    last_check: Optional[HealthResult]
    circuit: CircuitBreaker

---

اجزای داخلی

کلاس‌های زیر توسط "RelayManager" استفاده می‌شوند و معمولاً نیازی به استفادهٔ مستقیم از آن‌ها نیست:

- "HealthChecker"
- "PolicyEngine"
- "RateLimiter"
- "CircuitBreaker"

---

توسعه و مشارکت

پیشنهادها، باگ‌ریپورت‌ها و مشارکت‌های شما با خوشحالی پذیرفته می‌شود.

مخزن پروژه: "https://github.com/shanduzgil/netrelay"

«این آدرس را با مخزن واقعی خود جایگزین کنید.»

راه‌اندازی پروژه در محیط محلی

git clone <repo-url>
cd netrelay
pip install -e ".[dev]"
pytest

---

مجوز

این پروژه تحت مجوز MIT منتشر شده است.
متن کامل مجوز در فایل "LICENSE" قرار دارد.
