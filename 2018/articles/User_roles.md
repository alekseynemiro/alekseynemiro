# User roles in Web applications

When developing web projects, it is often necessary to differentiate the level of user permissions. User roles (user groups) will help to solve the problem. In this post, I will share my experience of creating user roles in web projects.

## Data structure

Typically, three tables store users and roles:

1. Users (userId, login, password, email, etc);
2. Roles (roleId, roleName);
3. Users and roles relationships (userId, roleId).

This data structure is the most optimal and easily scalable.

The number of tables and fields can be any, depending on the project.

## Users

The `Users` role is controversial.

In various projects, I often see the `Users` role. However, there is no use of this role. The database stores unnecessary links. It is inconvenient to use, and this is additional overheads. In the past, I did the same.

Now I usually adhere to the rule that any registered user by default is a user and does not need to add it to the `Users` role.

Of course, we can use the `Users` role to check the registration confirmation. However, I think that for this purpose it is more convenient to use a separate flag in the users’ table than the role. When working with data directly in the database, this solution will save a lot of time.

There may be situations when the site needs to limit the functionality for confirmed (approved) and unconfirmed users. I.e., in this case, the `Users` role acts as a sign of confirmation of the account. Using a separate role will be a more convenient solution than using a flag since we can use standard role-based access control filters. But this is a specific task. Most sites do not need to put each user in the `Users` role. If there is such a need in the future, then it will not be difficult to add a new role.

In any case, the `Users` role can and should exist in the project, at least to preserve order, even if there is no explicit addition of all users to this role.

## Administrators

The `Administrators` role is usually standard and gives users access to the functionality for managing the site. There is nothing more to add.

## System

In large projects, it may need to perform some tasks and display results in logs or in the interface. The `System` role is well suited for these purposes.

For example, the project may have the following system users:

* Log Cleaner
* Cache Cleaner
* File Checker
* Payment Manager
* Ticket Manager
* etc.

## Developers

The `Developers` role in the first place may limit access to the test site and/or pages under development.

In addition, the `Developers` role can be useful for the output on the site of technical information or warnings for developers. The correct operation of certain functions may not be possible to verify in the test mode, only in real conditions. For example, when developing an air ticket issuance system, in some cases, real fines may arise, and the company may incur financial losses. For developers, we can display a warning about possible consequences and conditions of occurrence.

In addition, the `Developers` role is useful when it is impossible to replicate the problem in a test environment.

However do not abuse the `Developers` role and do not have a strong impact on the user interface, otherwise, all the benefits will turn into additional problems and misunderstandings.

## Testers

The `Testers` role is useful for providing test access.

For example, when integrating with social networks (such as Facebook, VK, etc.); we may need to give test access for sites representatives so that they can verify the correct operation of the system. If it is necessary to do it on the published site, then ordinary users should not notice anything. The `Testers` role allows solving this problem.

## Blocked

The account table for blocking users usually uses the corresponding flag (bit field). But why not use a separate role for this? After all, not every user will be blocked on the site. Most likely, the percentage of blocked users will be insignificant, especially with the proper organization of the registration system.

## Conclusion

A set of roles can be any, depending on the needs of a particular project. I listed the main roles that I use in almost every project. If you have something to add, it would be interesting to see.

---
Aleksey Nemiro  
2018-03-01

https://alekseynemiro.medium.com/user-roles-in-web-applications-ac524f8b6bcd
