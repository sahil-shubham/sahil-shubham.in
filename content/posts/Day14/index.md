+++
title="Day-14, LocalStorage need to kept in check"
date=2021-08-13
+++

I like localStorage. They have always been the goto place whenever I needed to store anything easily and/or momentarily. Sometimes as a check for first time visitors, sometimes as a **low grade** state managment or sometimes when I don't want to ask the server for something frequently.

This blogpost is about an unexpected issue I ran into while optimizing the search results speed on Neera.

## Preface

Neera offers custom profiles which make search through particular websites, or personal documents easier. Anyone can make any number of personalized tabs to shift through different sources quickly to filter search results. One just need to signUp on Neera. But this wasn't how it always was.

During the early stages of Neera, profiles weren't customizable and were same for everyone. These were set on the first render and were stored in the localStorage from where they were later taken from. We later implemented signUp on our platform which changed the workflow of getting a result to the following:

(Let's assume Neera is the default search engine)

1. You enter a query directly in address bar
2. We send a request to backend asking for profiles
3. It checks if any cookies are attached to the request which determine whether the user is logged in or not

- If they are logged in, backend sends another request to the database ([Supabase](https://app.supabase.io) here) and sends back the response
- If they aren't logged in, backend sends the default profiles which is an object stored in backend

4. Once the frontend receives the profiles, it sets them as the state and a selected profile and tab state are set (this is the profile/tab that query is initially searched in)
5. Now it sends a request to the backend with the query and the selected tab and profile which lets it know what results it should filter or API should it use (in some cases we use a service specific API rather than filtering search results by domain, e.g. GitHub)
6. The backend uses Google/Bing API and send back the results which are later rendered

As you can see there is a lot happening, our searches were better but they would always be slower than other search engines because they start their process at step 5.

I wanted us to have a middle ground and hence tried to keep the profiles client side itself, because they would only change when an user modifies them at which point I would update them in both the backend and frontend. This would help in getting us directly at step 4. Decreasing 1-2 request calls.

## The issue

I implemented the above and tested cases where someone could be opening the website for the first time,or multiple times, how it looks on network throttling or if the profiles were present in localStorage but empty. Everything seemed fine.

Pushed and deployed, really happy with the results we continued with other work for the next few weeks. Only to be contacted by one of the first users who had trouble opening the site which they hadn't since a few months.

Fortunately for us, they attached the screenshot with their console open (We need more people like this). Somehow the profiles were undefined, but I had implemented a check for this.

Did I make a mistake? Umm, most probably. Better question would be, what mistake did we do?

Remember how I mentioned Neera didn't use to have user logins and depended on localStorage for profiles? Well coincidentally I stored profiles in the same key name we used to, but in a different format ofcourse, as things have changed. Technically the user above did have the profiles but in a different format :|

## How to fix it? and did we do it?

We looked at our options:

- We can have a version key value pair in anything we are storing so that we can compare if the schema/structre of something is old or new
- On new reloads we can delete and set profiles, this would remove everything old and new and start afresh
- Cookies are good in this case, as one can set their expiry
- We can change the keyname for storing profiles

We went with the last option, as it requires less work as well as less uncertainty about whether they would work in every case.

If one is constantly dependant on localStorage for their needs. They need to keep a check on it. So that no one whether old or new faces any issue with a new feature implementation. We were fairly lucky because of an user informing us, but there can be other users who never got to see how far everything has come due to this.

Anyways, every cloud has a silver lining :)

![trade offer](trade_offer.jpg)
