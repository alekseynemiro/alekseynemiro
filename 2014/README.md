# 2014

2014 was the year of cloud technologies, packages, and DBMS.

I got rid of the physical server and rented a cloud server from one of the first Russian cloud hosting companies.
It was really cool. Server parameters could be changed at any time.

I tried to reduce server costs and on average paid about $75-$100 per month.
This was less than $130 for a physical server.
However, I did not have enough server resources; the server was constantly overloaded.

Since cloud technologies were just emerging, there were many problems with reliability.
The number of incidents exceeded all acceptable limits.

At work, we also used cloud technologies. More precisely, I started using them because we did it at work.

In 2014 I released my first [NuGet](https://www.nuget.org/profiles/aleksey.nemiro) packages.

The most popular project was [Nemiro.OAuth](https://github.com/nemiro-net/nemiro.oauth).
Is a class library for authorization via OAuth protocol in .NET Framework.

The library not only made it possible to work with the OAuth protocol but also made it possible to obtain basic information about users from providers.
It was equally simple and convenient in both web projects and Windows Forms applications.

I also released the [Nemiro.Convertion](https://github.com/nemiro-net/nemiro.convertion) library that made it easier to convert types in .NET Framework projects.

In addition, I released the [Nemiro.Data](https://github.com/nemiro-net/nemiro.data.dll) package, which was a set of helper methods for working with the SQL Server DBMS, and also contained my ORM (Object-Relational Mapping) implementation.
The library came with a GUI program that allowed the user to generate C# or VB .NET classes from database tables.
I have used this library in all the projects I have ever worked with.

I continued to work on a fully automated web project for searching and booking airline tickets, which we received [last year](../2013).
I mostly worked on the backend. However the project needed serious frontend work, and a separate frontend team was brought in for this.
The frontend team used completely new methods for working with client-side code, style files, and image files.
This is how I got my first experience with SASS/SCSS (Syntactically Awesome Style Sheets).

We also connected the project to aggregators such as Momondo, AviaSales, Yandex, etc.
Each aggregator provided hundreds of requests per minute.
Each request generated an average of 2 MB of data, and the raw data was tens of times larger.
Due to the fact that in my practice I have developed similar systems using more modest technical resources, we did not have any problems with performance.

This year I also started developing a project - a database of travel objects.
It was a 90% SQL Server level project, with database programming using T-SQL.
The main goal of the project was to create a unified data storage and its subsequent delivery to clients.

## Solutions for .NET Framework

:star: [Nemiro.OAuth v1.1-v1.5](https://github.com/nemiro-net/nemiro.oauth/tree/v1.5)  
:star: [Nemiro.Convertion v2.1](https://github.com/nemiro-net/nemiro.convertion/tree/v2.1)  
:star: [Nemiro.Data](https://github.com/nemiro-net/nemiro.data.dll)

## Articles

:page_facing_up: [Авторизация по протоколу OAuth в проектах .NET Framework](articles/OAuth.md)

---
[< 2013](/2013) &nbsp; **2014** &nbsp; [2015 >](/2015)

---
Aleksey Nemiro  
2023-09-13
