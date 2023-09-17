# 2016

In 2016, I started experimenting with using Debian and Mono in production environment.

I deployed the [FoxTools.ru API](../2015/assets/foxtools.md) on Debian.

Despite the fact that the hardware was more powerful than on a Windows server, the operating efficiency was low.
The main problem was the slow FastCGI.

In the spring, I released my own tools to configure and deploy a Debian server - [HTAN](https://github.com/adminstock/htan).

I also released [SmallServerAdmin](https://github.com/adminstock/ssa), a web-based server control panel.
The backend was written in PHP using Bash inserts. For the frontend, I used AngularJS and TypeScript.

In the middle of the year, I rented a Windows virtual server in Germany.
It was a server with 8 GB RAM and 200 GB SSD. For only 20 euros per month!

But I didn't give up on Debian. I liked the traffic management capabilities of Nginx.
Also, I liked that the cost of Debian-based servers was low with good characteristics.

In 2016, together with Arimsoft, we began developing a joint project on a partnership basis.
This was a project to implement a unified platform for communicating with users through various messaging systems.  
And also with the ability to create chatbots.

On the backend side, the project was written in ASP.NET Web API, C#. For data, we used the PostgreSQL DBMS.

Technically, it was a large and complex project.

The most interesting and difficult integration was with WhatsApp because WhatsApp did not have an API.

I used the WhatsApp web interface to understand how it works. I spent many days trying to figure out confusing JavaScript code and implement it in C#.
As a result, we have implemented integration with WhatsApp.

On the frontend side, I created SAP (single-page application).
I used Webpack to bundle the client code.

This was my first React application.

We used PostgreSQL because of its high performance, scalability, and, of course, it's free.
PostgreSQL implemented some functionality that I tried to implement in SQL Server to get better performance.
For example, dividing tables into parts.

I used PL/pgSQL to write stored procedures and functions.

We used the Bot Framework as a reference point. But the project architecture was quite flexible and we were not tied to any specific platform.

## Solutions for .NET Framework

:star: [Nemiro.OAuth v1.11-v1.12](https://github.com/nemiro-net/nemiro.oauth/tree/v1.12)  
:star: [Nemiro.Convertion v2.3](https://github.com/nemiro-net/nemiro.convertion/tree/v2.3)

## Solutions for Debian/Ubuntu

:star: [SmallServerAdmin (SSA)](https://github.com/adminstock/ssa)  
:star: [HTAN](https://github.com/adminstock/htan)  
:star: [HTAN.Runner](https://github.com/adminstock/htan-runner)  
:star: [ApacheConf.PHP](https://github.com/phperry/ApacheConf.PHP)  
:star: [NginxConf.PHP](https://github.com/phperry/NginxConf.PHP)

## Articles

:page_facing_up: [Users Authorization through OAuth in .NET Framework](articles/OAuth.md)

---
[< 2015](/2015) &nbsp; **2016** &nbsp; [2017 >](/2017)

---
Aleksey Nemiro  
2023-09-15
