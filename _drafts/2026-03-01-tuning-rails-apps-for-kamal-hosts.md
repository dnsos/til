A server with 2vCPU can have `WEB_CONCURRENCY` set to 2 for 2 Puma processes:

https://guides.rubyonrails.org/tuning_performance_for_deployment.html#practical-implications

But if the app is IO-focussed (database calls, external API calls, etc.), this is not necessarily a good idea. The app may not need the CPUs because the operations are not CPU-focussed.

---

If that server hosts other services (such as a Kamal database accessory), we should leave room and potentially stick with the default which is one Puma process.

https://guides.rubyonrails.org/tuning_performance_for_deployment.html#puma

Note that 1 vCPU does not necessarily mean 1 CPU with some cloud hosters, so keep that in mind when tuning for production.
