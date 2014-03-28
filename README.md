coord
=====

An application and server lifecycle coordination tool.

Manages applications and their lifecycle including various content providers such as Pulp and Puppet. Allows for automatic promotion and continious integration of content through an applications lifecycles.

Allows multiple teams to manage modules and push those modules through lifecycles of applications that are using their modules.

* Full role based access
* Integration with Foreman for server buils
* Integration with MCollective for server tasks
* Integration with Pulp for rpm package deployments
* Puppet-librarian for puppet environment complialation
* Allow non-team members to subscribe to applications for change notification
* Integration with Jira (approval workflows ???)
* Full Rest api (duh)

Why coord?
==========
Working in a an environment where different teams manage different aspects of a system it can be hard to allow those teams to properly manage their piece of a system.  Those tools work great if you can grant full control of a system to whoever needs to manage them.  For basic applications this works fine, but what about systems with PII or PCI data? How do you let an infrastructure team keep control of basic operating system functions such as dns resolution, mail configurations, and authentication mechanisms. Then how do you let security teams manage tripwire and audit configurations?  Lastly, how do you let application developers manage their applications?  How do you do all this while keeping seperation of roles and permissions?  That is gola of coord.  Coord lets you coordinate the management of your systems the way you want to.

Example Structures
==================

#### Content Views
* /api/v1/contentviews [GET, POST]

```javascript
[
  {
    "name": "rhel6-x86_64",
    "teams": ['infrastructure'],
    "repositories": [
      "rhel6-updates-x86_64",
      "rhel6-epel-x86_64",
      "rhel6-optional-x86_64",
    ],
  },
  {
    "name": "puppet-infs",
    "teams": ['infrastructure'],
    "repositories": [
      "infs-resolv",
      "infs-motd",
    ],
  },
  {
    "name": "puppet-my-app",
    "teams": ['Dev Group 1'],
    "repositories": [
      "devgroup1-app1",
    ],
  }
]
```
* /api/v1/contentviews/rhel6-x86_64 [GET, PUT, DELETE]
* /api/v1/contentviews/rhel6-x86_64/applications [GET]

   This should return any applications which actively this contentview
* /api/v1/contentviews/rhel6-x86_64/changegroups [GET]

   This should return any changegroups which have this contentview as active or pending by default
   
* /api/v1/contentviews/rhel6-x86_64/changesets [GET]

   This should return any changesets which contain this contentview 

* /api/vi/contentviews/rhel6-x86_64/makechangeset [POST]

   This should submit a changeset building tasks.  If no applications are specified, it should create a changeset for all, otherwise applications should be specified to limit scope. Return would be a task id

#### Repositories
* /api/v1/repositories [GET, POST]

```javascript
[
  {
    "name": "rhel6-updates-x86_64",
    "provider": "pulp",
    "team": "Infrastructure"
    "pakages": [
      {
        "name": "kernel",
        "version": "2.6.32",
        "release": "345.5.el6_5",
        "epoch": "0",
        "arch": "x86_64",
        "file": "kernel-2.6.32-345.5.el6_x86_64"
      }
    ]
  },
  {
    "name": "infs-resolv",
    "provider": "puppet",
    "team": "Infrastructure"
    "pakages": [
      {
        "name": "resolv",
        "version": "1.0",
      }
    ]
  }
]
```
* /api/v1/repositories/rhel6-updates-x86_64 [GET, PUT, DELETE]
* /api/v1/repositories/rhel6-updates-x86_64/applications [GET]

   This should return any applications which have content-views that contain this repository
   
* /api/v1/repositories/rhel6-updates-x86_64/changegroups [GET]

   This should return any changegroups which have content-views that contain this repository
   
* /api/v1/repositories/rhel6-updates-x86_64/changesets [GET]

   This should return any changesets which contain this repository

* /api/vi/repositories/rhel6-updates-x86_64/makechangesets [POST]

   This should submit a changeset building tasks.  If no applications specified, it should create a changeset for all, otherwise applications should be specified to limit scope. Return would be a task id

#### Applications
* /api/v1/applications [GET, POST]

```javascript
{
  "name": "my-application",
  "teams": [
    {
      "name": "Infrastructure",
      "role": "Administrator",
    },
    {
      "name": "Dev Group 1",
      "role": "Developer",
    },
    {
      "name": "Management",
      "role": "Watcher",
    }
  ],
  "content-views": [
    {
      "name": "rhel6-x86_64",
      "changegroups": [
        {
          "name": "dev",
          "status": "applied",
          "type": "scheduled",
          "changesets": [
            {
              "changeset-20140203",
              "status": "active",
              "deployment_date": "2013-02-03 0500-0800",
              "roles": [],
            },
            {
              "changeset-20140301",
              "status": "scheduled",
              "deployment_date": "2013-03-03 0500-0800",
              "roles": [],
            }
          ]
          "schedule": {
            "everyType": "Month",
            "everyNum": 1,
            "starting": "Monday",
            "after": "2014-03-01",
            "time": "0500-0800",
          }
        }
      ]
    }
  ],
  "lifecycles": [
    {
      "name": "Development",[
      "changegroups": [
        {
          "name": "dev",
          "previous": "",
          "content-views": [
            {
              "name": "rhel6-x86_64",
              "changeset": "changeset-20140203",
              "roles": [],
            }
          ]
          "servers": [
            {
              "name": "dev-web1",
              "role": "web",
            }
            {
              "name": "dev-db1",
              "role": "db",
            }
          ]
        }
      ]
    }
  ]
}
```
* /api/v1/applications/my-application [GET, PUT, DELETE]
* /api/v1/applications/my-application/makechangegroup [POST]

   This should submit a changeset creation task.  If no content-views are specified, a changeset will be created from all content-views.  The changeset will be scheduled on the first changegroup.
   
* /api/v1/applications/my-application/lifecycles [GET, POST]
* /api/v1/applications/my-application/changegroups [GET, POST]
* /api/v1/applications/my-application/changegroups/dev1 [GET, PUT, DELETE]
* /api/v1/applications/my-application/changegroups/dev1/promote [POST]
* /api/v1/applications/my-application/contentviews [GET, POST]
* /api/v1/applications/my-application/changesets [GET, POST]

#### Services
* /api/v1/services [GET, POST]

```javascript
[
  {
    "name": "app-service",
    "application": "my-application",
    "endpoints": [
      {
        "segment": "dc1",
        "lifecycle": "Production",
        "data": "http://dc1.my-application.com"
      }
    ]
  }
]
```
#### Changesets
* /api/v1/changesets [GET, POST]

```javascript
[
  {
    "name": "changeset-20140208",
    "application": "my-application",
    "status": "pending",
    "content-view": "rhel6-x86_64",
    "repositories": [
      {
        "name": "rhel6-updates-x86_64",
        "packages": [ {} ]
      }
    ]
  }
]
```

* /api/v1/changesets/changeset-20140208/promote [POST]

  This should submit a promotion task for the given changegroup.  If no destination changegroup is specified, it will be promoted to all changegroups

#### Teams
* /api/v1/teams [GET, POST]

```javascript
[
  {
    "name": "Infrastructure",
    "members": [
      {
        "email": "john@doe.com"
      }
    ]
  }
]
```

#### Users
* /api/v1/users [GET, POST]

```javascript
[
  {
    "name": "John Doe",
    "email": "john@doe.com"
  }
]
```


