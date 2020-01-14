# Groza

_Fast and reliable reactivity between user and service data._

Groza goal to framework transparent and reliable reactivity UX 
between user and service data.

Reactivity of data changes from service.

Automatically saved pattern.
Trivial Undo (Ctrl/Cmd+Z) implementation in UX.

## Content

Preface
Principles
Multiply connections
Single connections

## Preface

Document describes protocol specification between client-side e.g. user browser
and server-side e.g. web service.

Requrement for storages to have _watch_ data functions. PostgreSQL, Redis 
and some other have this out of box. Or service must be only point for data
changing.

It's primarily bidirectional protocol. Both sides client and server can send 
messages.
Which also can be achieved by HTTP+webhooks, but currently concentrates 
on Websocket.

Subscription for any changes goes the same way you request data. When the data
changes you will receive this update wright now.

More light backend side, which allows almost direct access to database, 
but also secured by permissions.

We use JSON format in examples and implementantion, but it is not required.

    {
        "type": "sub",
        "queryId": "..."
    }
    
Response:   
    
    {
        "type": "data",
        "responseQueryId": "..."
    }

On client-side:

    {
        "subscription": {
            allTemplates: { visor: "Template" },
        }
    }
    
    
Use _visor_ word. It was like supervisor. But supervisor is too long. super 
has different meaning in Python. 
So now it's visor.

On server-side (Python3):

    class Template(GrozaVisor):
        table = 'templates'
        primary_key = 'id'
        
        
## Subscriptions

Subscribe data.
    
    allTemplates: { visor: "Template" }
   
Request data once and don't receive any updates.
    
    {
        "subscription": { ... },
        "once": {
            allTemplates: { visor: "Template" },
        }
    }
    
## Result data

All data we got are flat. No hierarchy objects are made in results.

    {
        "responseQueryId": "82fc533a-36d2-446b-8d74-a6f287257ef6",
        "type": "data",
        "data": {
            "templates": {
                1: {"id": 1, "content": "# 1", "isEnabled": true,
                    "lastUpdatedBy": 1}, 
                2: {"id": 2, "content": "# 2", "isEnabled": true,
                    "lastUpdatedBy": 1}
            }
        },
        "sub": {
            "allTemplates": {
                "status": "ok",
                "dataField": "templates",
                "ids": [1, 2],
                "fromSub": {}
            }
        }
    }

Simpler cache. Easily add, update or remove data contents in user browser.
Cache data in user browser for few last pages, so user can absolutely fast 
get back and get correct page.

For example in VueJS with Vuex Store it looks like this:

    getters: {
        profiles: (state) => state.profileIds.length ? 
            state.profileIds.map(
                profileId => state.data.profiles[profileId]) 
            : [],
    }

And in component we have computed field `profiles` (Typescript)

    get profiles() {
        return this.$store.getters.profiles.sort(function(a, b) {
          return a.title - b.title;
        });
    }
    
 As you can see we apply custom sort for data too.

