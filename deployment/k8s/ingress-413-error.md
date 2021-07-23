---
title: ingress-413-error
date: 2021-01-31 13:00:00
tags: '部署'
categories:
  - ['部署', 'k8s']
permalink: ingress-413-error
photo:
---

https://stackoverflow.com/questions/49918313/413-error-with-kubernetes-and-nginx-ingress-controller

# [413 error with Kubernetes and Nginx ingress controller](/questions/49918313/413-error-with-kubernetes-and-nginx-ingress-controller)

[Ask Question](/questions/ask)

Asked 2 years, 9 months ago

Active [5 months ago](? lastactivity "2020-08-15 13:08:20Z")

Viewed 16k times

23

5

[](/posts/49918313/timeline "Show activity on this post.")

I'm trying to change the `client_max_body_size` value, so my nginx ingress will not return 413 error.

I've tested few solutions.
Here is my test config map:

```
kind: ConfigMap
apiVersion: v1
data:
  proxy-connect-timeout: "15"
  proxy-read-timeout: "600"
  proxy-send-timeout: "600"
  proxy-body-size: "8m"
  hsts-include-subdomains: "false"
  body-size: "64m"
  server-name-hash-bucket-size: "256"
  client-max-body-size: "50m"
metadata:
  name: nginx-configuration
  namespace: ingress-nginx
  labels:
    app: ingress-nginx
```

These changes has no effect at all, after loading it, in the nginx controller log I can see the information about reloading config map, but the values in nginx.conf are the same:

```
root@nginx-ingress-controller-95db685f5-b5s6s:/# cat /etc/nginx/nginx.conf | grep client_max
                        client_max_body_size                    "8m";
                        client_max_body_size                    "1m";
                        client_max_body_size                    "1m";
                        client_max_body_size                    "1m";
                        client_max_body_size                    "1m";
                        client_max_body_size                    "1m";
                        client_max_body_size                    "1m";
                        client_max_body_size                    "1m";
                        client_max_body_size                    "1m";
                        client_max_body_size                    "1m";
                        client_max_body_size                    "1m";
                        client_max_body_size                    "1m";
                        client_max_body_size                    "1m";
```

My nginx-controller config uses this image: quay.io/kubernetes-ingress-controller/nginx-ingress-controller:0.13.0

How can I force the nginx to change the value? I need to change it globally, for all my ingresses.

[nginx](/questions/tagged/nginx "show questions tagged 'nginx'") [kubernetes](/questions/tagged/kubernetes "show questions tagged 'kubernetes'")

﻿

[Share](/q/49918313 "Short permalink to this question")

[Improve this question](/posts/49918313/edit)

Follow

asked Apr 19 '18 at 10:06

[

! [](https://www.gravatar.com/avatar/75896ac961d6cfc55d21d297cc3b11b3? s=32&d= identicon&r= PG)

](/users/855472/djent)

[Djent](/users/855472/djent) Djent

1,09944 gold badges2121 silver badges4545 bronze badges

[add a comment](# "Use comments to ask for more information or suggest improvements. Avoid answering questions in comments.") | [](# "expand to show all comments on this post")

## 5 Answers 5

[Active](/questions/49918313/413-error-with-kubernetes-and-nginx-ingress-controller? answertab= active#tab-top "Answers with the latest activity first") [Oldest](/questions/49918313/413-error-with-kubernetes-and-nginx-ingress-controller? answertab= oldest#tab-top "Answers in the order they were provided") [Votes](/questions/49918313/413-error-with-kubernetes-and-nginx-ingress-controller? answertab= votes#tab-top "Answers with the highest score first")

56

[](/posts/49918432/timeline "Show activity on this post.")

You can use the [annotation](https://github.com/kubernetes/ingress-nginx/blob/master/docs/user-guide/nginx-configuration/annotations.md#custom-max-body-size) `nginx.ingress.kubernetes.io/proxy-body-size` to set the max-body-size option right in your Ingress object instead of changing a base ConfigMap.

Here is the example of usage:

```
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: my-app
  annotations:
    nginx.ingress.kubernetes.io/proxy-body-size: "50m"
...
```

﻿

[Share](/a/49918432 "Short permalink to this answer")

[Improve this answer](/posts/49918432/edit)

Follow

[edited Apr 26 '18 at 11:39](/posts/49918432/revisions "show all edits to this post")

[

! [](https://www.gravatar.com/avatar/d12a4eb0f635899a630b71235dc43cf8? s=32&d= identicon&r= PG&f=1)

](/users/3906217/adri%c3%a1n)

[Adrián](/users/3906217/adri%c3%a1n)

2,7221313 silver badges2121 bronze badges

answered Apr 19 '18 at 10:12

[

! [](https://www.gravatar.com/avatar/28ba912aad9ed79488592b6d4e95e59a? s=32&d= identicon&r= PG&f=1)

](/users/5937420/anton-kostenko)

[Anton Kostenko](/users/5937420/anton-kostenko) Anton Kostenko

5,70411 gold badge1616 silver badges2828 bronze badges

1

*   1

    I wanted to set it globally, but actually this approach is better as I have my yaml files within the project repository. So it won't be lost. Thanks. – [Djent](/users/855472/djent "1,099 reputation") Apr 19 '18 at 11:13


[add a comment](# "Use comments to ask for more information or suggest improvements. Avoid comments like "+1" or "thanks".") | [](# "expand to show all comments on this post")

10

[](/posts/52140974/timeline "Show activity on this post.")

To set it globally, this [configmap.md](https://github.com/kubernetes/ingress-nginx/blob/master/docs/user-guide/nginx-configuration/configmap.md#proxy-body-size) documentation might be helpful. Turns out the variable to set is `proxy-body-size`, not `client-max-body-size`.

When you deploy the helm chart, you can set `--set-string controller.config.proxy-body-size="4m"`.

﻿

[Share](/a/52140974 "Short permalink to this answer")

[Improve this answer](/posts/52140974/edit)

Follow

answered Sep 2 '18 at 20:57

[

! [](https://i.stack.imgur.com/Q6ybk.jpg? s=32&g=1)

](/users/860756/sojen)

[SoJeN](/users/860756/sojen) SoJeN

32933 silver badges77 bronze badges

[add a comment](# "Use comments to ask for more information or suggest improvements. Avoid comments like "+1" or "thanks".") | [](# "expand to show all comments on this post")

1

[](/posts/59971397/timeline "Show activity on this post.")

Update:

I have been experiencing the same problem and no solutions were working. After reading through countless blogs and docs that all had the same suggested solution I found that they have changed the naming convention.

It is no longer denoted by "proxy-body-size" or this just never works for me.

link below shows that the correct configmap variable to use is "client-max-body-size"

[https://docs.nginx.com/nginx-ingress-controller/configuration/global-configuration/configmap-resource/](https://docs.nginx.com/nginx-ingress-controller/configuration/global-configuration/configmap-resource/)

﻿

[Share](/a/59971397 "Short permalink to this answer")

[Improve this answer](/posts/59971397/edit)

Follow

answered Jan 29 '20 at 16:22

[

! [](https://www.gravatar.com/avatar/b0be0d191d4817cd309b46804b57f888? s=32&d= identicon&r= PG)

](/users/2277250/joe-roberts)

[Joe Roberts](/users/2277250/joe-roberts) Joe Roberts

5966 bronze badges

2

*   1

    For those reading, this comment applies when using the NGINX-issued ingress controller. If you are using the Kubernetes-issued NGINX controller the property name is still `proxy-body-size`. You can tell this by the annotation key which will be `nginx.ingress.kubernetes.io/xxxxx` for Kubernetes-issued rather than `nginx.org/xxxxx` which is NGINX-issued. – [James G](/users/1196415/james-g "1,239 reputation") Oct 12 '20 at 5:07

*   I encourage the folks at Nginx and Kubernetes to get together and develop a common spec – [code\_monk](/users/977083/code-monk "6,342 reputation") Nov 24 '20 at 3:14


[add a comment](# "Use comments to ask for more information or suggest improvements. Avoid comments like "+1" or "thanks".") | [](# "expand to show all comments on this post")

0

[](/posts/60625626/timeline "Show activity on this post.")

I have tried both proxy-body-size and client-max-body-size on the configmap and did a rolling restart of the nginx controller pods and when I grep the nginx.conf file in the pod it returns the default 1m. I am trying to do this within Azure Kubernetes Service (AKS). I'm working with someone from their support. They said its not on them since it appears to be a nginx config issue.

The weird hting is we had other clusters in Azure that this wasnt an issue on until we discovered this with some of the newer deployments. The initial fix they came up with is what is in this thread but it just refuses to change.

Below is my configmap:

```
# Please edit the object below. Lines beginning with a '#' will be ignored,
# and an empty file will abort the edit. If an error occurs while saving this file will be
# reopened with the relevant failures.
#
apiVersion: v1
data:
  client-max-body-size: 0m
  proxy-connect-timeout: 10s
  proxy-read-timeout: 10s
kind: ConfigMap
metadata:
  annotations:
    control-plane.alpha.kubernetes.io/leader: '{"holderIdentity":"nginx-nginx-ingress-controller-7b9bff87b8-vxv8q","leaseDurationSeconds":30,"acquireTime":"2020-03-10T20:52:06Z","renewTime":"2020-03-10T20:53:21Z","leaderTransitions":1}'
  creationTimestamp: "2020-03-10T18:34:01Z"
  name: ingress-controller-leader-nginx
  namespace: ingress-nginx
  resourceVersion: "23928"
  selfLink: /api/v1/namespaces/ingress-nginx/configmaps/ingress-controller-leader-nginx
  uid: b68a2143-62fd-11ea-ab45-d67902848a80
```

After issuing a rolling restart: kubectl rollout restart deployment/nginx-nginx-ingress-controller -n ingress-nginx

Grepping the nginx ingress controller pod to query the value now reveals:

```
kubectl exec -n ingress-nginx nginx-nginx-ingress-controller-7b9bff87b8-p4ppw cat nginx.conf | grep client_max_body_size
            client_max_body_size                    1m;
            client_max_body_size                    1m;
            client_max_body_size                    1m;
            client_max_body_size                    1m;
            client_max_body_size                    21m;
```

Doesnt matter where I try to change it. On the configmap for global or the Ingress route specifically.......this value above never changes.

﻿

[Share](/a/60625626 "Short permalink to this answer")

[Improve this answer](/posts/60625626/edit)

Follow

[edited Mar 10 '20 at 20:56](/posts/60625626/revisions "show all edits to this post")

answered Mar 10 '20 at 20:41

[

! [](https://lh6.googleusercontent.com/-7PizNS816ZY/AAAAAAAAAAI/AAAAAAAAAiU/mgyOGysRdtE/photo.jpg? sz=32)

](/users/11771838/hurell-lyons)

[Hurell Lyons](/users/11771838/hurell-lyons) Hurell Lyons

111 bronze badge

1

*   I can confirm this, we are running in exactly the same problem. AKS version 1.18.4. – [Patrick P.](/users/2363917/patrick-p "99 reputation") Aug 25 '20 at 15:12


[add a comment](# "Use comments to ask for more information or suggest improvements. Avoid comments like "+1" or "thanks".") | [](# "expand to show all comments on this post")

0

[](/posts/63426238/timeline "Show activity on this post.")

I am able to fix it by redeploying the nginx-ingress controller after modifying the configmap.

```
kubectl patch deployment your_deployment -p "{\"spec\": {\"template\": {\"metadata\": { \"labels\": {  \"redeploy\": \"$(date +%s)\"}}}}}"
```

﻿

[Share](/a/63426238 "Short permalink to this answer")

[Improve this answer](/posts/63426238/edit)

Follow

[edited Aug 15 '20 at 13:08](/posts/63426238/revisions "show all edits to this post")

[

! [](https://i.stack.imgur.com/EYylF.jpg? s=32&g=1)

](/users/1839482/arghya-sadhu)

[Arghya Sadhu](/users/1839482/arghya-sadhu)

25.9k99 gold badges3232 silver badges5454 bronze badges

answered Aug 15 '20 at 12:51

[

! [](https://lh4.googleusercontent.com/-Ojet63n0MEg/AAAAAAAAAAI/AAAAAAAAAAA/AMZuuckmRwcrmhmRtpI6Pxe1KR8WdoJLXg/photo.jpg? sz=32)

](/users/13559351/selvam-elangovan)

[Selvam Elangovan](/users/13559351/selvam-elangovan) Selvam Elangovan

1

[add a comment](# "Use comments to ask for more information or suggest improvements. Avoid comments like "+1" or "thanks".") | [](# "expand to show all comments on this post")



## Your Answer



Thanks for contributing an answer to Stack Overflow!

*   Please be sure to _answer the question_. Provide details and share your research!

But _avoid_ …

*   Asking for help, clarification, or responding to other answers.
*   Making statements based on opinion; back them up with references or personal experience.

To learn more, see our [tips on writing great answers](/help/how-to-answer).

Draft saved

Draft discarded



### Sign up or [log in](/users/login? ssrc= question_page&returnurl= https%3a%2f%2fstackoverflow.com%2fquestions%2f49918313%2f413-error-with-kubernetes-and-nginx-ingress-controller%23new-answer)

Sign up using Google

Sign up using Facebook

Sign up using Email and Password

  Submit

### Post as a guest

Name

Email

Required, but never shown

<h3 class="grid--cell fs-title"> Post as a guest</h3> <div class="grid--cell"> <div class="grid gs4 gsy fd-column"> <label class="s-label" for="display-name"> Name</label> <div class="grid ps-relative"> <input class="s-input" id="display-name" name="display-name" maxlength="30" type="text" value="" tabindex="105" placeholder="" /> </div> </div> </div> <div class="grid--cell"> <div class="grid gs4 gsy fd-column"> <div class="grid--cell"> <div class="grid gs2 gsy fd-column"> <label class="grid--cell s-label" for="m-address"> Email</label> <p class="grid--cell s-description"> Required, but never shown</p> </div> </div> <div class="grid ps-relative"> <input class="s-input js-post-email-field" id="m-address" name="m-address" type="text" value="" size="40" tabindex="106" placeholder="" /> </div> </div> </div>

Post Your Answer Discard

By clicking "Post Your Answer", you agree to our [terms of service](https://stackoverflow.com/legal/terms-of-service/public), [privacy policy](https://stackoverflow.com/legal/privacy-policy) and [cookie policy](https://stackoverflow.com/legal/cookie-policy)

## Not the answer you're looking for? Browse other questions tagged [nginx](/questions/tagged/nginx "show questions tagged 'nginx'") [kubernetes](/questions/tagged/kubernetes "show questions tagged 'kubernetes'") or [ask your own question](/questions/ask).

The Overflow Blog

*   [Sequencing your DNA with a USB dongle and open source code](https://stackoverflow.blog/2021/02/03/sequencing-your-dna-with-a-usb-dongle-and-open-source-code/)

*   [Podcast 310: Fix-Server, and other useful command line utilities](https://stackoverflow.blog/2021/02/05/podcast-310-fix-server-and-other-useful-command-line-utilities/)


Featured on Meta

*   [Opt-in alpha test for a new Stacks editor](https://meta.stackexchange.com/questions/360033/opt-in-alpha-test-for-a-new-stacks-editor)

*   [Visual design changes to the review queues](https://meta.stackexchange.com/questions/360198/visual-design-changes-to-the-review-queues)


[Visit chat](https://chat.stackoverflow.com/)

#### Linked

[

1

](/q/56780810 "Vote score (upvotes - downvotes)") [net:: ERR\_CONNECTION\_RESET Managed Kubernetes Digital Ocean Large Payloads](/questions/56780810/neterr-connection-reset-managed-kubernetes-digital-ocean-large-payloads? noredirect=1)

[

0

](/q/50203277 "Vote score (upvotes - downvotes)") [Openwhisk: Request Entity Too Large](/questions/50203277/openwhisk-request-entity-too-large? noredirect=1)

#### Related

[

1036

](/q/5009324 "Vote score (upvotes - downvotes)") [Node.js + Nginx - What now?](/questions/5009324/node-js-nginx-what-now)

[

1

](/q/49426772 "Vote score (upvotes - downvotes)") [Nginx server behind Nginx-Ingress controller](/questions/49426772/nginx-server-behind-nginx-ingress-controller)

[

13

](/q/52782415 "Vote score (upvotes - downvotes)") [nginx-ingress config map snippets being ignored by the nginx.conf](/questions/52782415/nginx-ingress-config-map-snippets-being-ignored-by-the-nginx-conf)

[

0

](/q/56120575 "Vote score (upvotes - downvotes)") [nginx-ingress returning 404 with multiple ingresses files](/questions/56120575/nginx-ingress-returning-404-with-multiple-ingresses-files)

[

3

](/q/59488401 "Vote score (upvotes - downvotes)") [annotation proxy-body-size doesn't work in kubernete nginx-ingress](/questions/59488401/annotation-proxy-body-size-doesnt-work-in-kubernete-nginx-ingress)

[

29

](/q/61365202 "Vote score (upvotes - downvotes)") [Nginx Ingress: service "ingress-nginx-controller-admission" not found](/questions/61365202/nginx-ingress-service-ingress-nginx-controller-admission-not-found)

[

0

](/q/61548102 "Vote score (upvotes - downvotes)") [NGINX proxy to Ingress Controller with Client Certificate Authentication](/questions/61548102/nginx-proxy-to-ingress-controller-with-client-certificate-authentication)

[

0

](/q/62657398 "Vote score (upvotes - downvotes)") [How to set max request body size in traefik ingress controller for kubernetes?](/questions/62657398/how-to-set-max-request-body-size-in-traefik-ingress-controller-for-kubernetes)

[

1

](/q/62925168 "Vote score (upvotes - downvotes)") [Kubernetes Nginx Ingress file upload returning 502](/questions/62925168/kubernetes-nginx-ingress-file-upload-returning-502)

#### [Hot Network Questions](https://stackexchange.com/questions? tab= hot)

*   [Finite-State Automata over a real-valued alphabet](https://cstheory.stackexchange.com/questions/48321/finite-state-automata-over-a-real-valued-alphabet)
*   [Pattern matching for repeated elements](https://mathematica.stackexchange.com/questions/239388/pattern-matching-for-repeated-elements)
*   [What are some fun projects for non-CS majors?](https://cseducators.stackexchange.com/questions/6777/what-are-some-fun-projects-for-non-cs-majors)
*   [How would a Steampunk voice synthesizer work?](https://worldbuilding.stackexchange.com/questions/195452/how-would-a-steampunk-voice-synthesizer-work)
*   [What is the formula unit surface area?](https://mattermodeling.stackexchange.com/questions/4279/what-is-the-formula-unit-surface-area)
*   [Is it immoral to advise PhD students in non-industry-relevant topics in middle-lower ranked universities?](https://academia.stackexchange.com/questions/162167/is-it-immoral-to-advise-phd-students-in-non-industry-relevant-topics-in-middle-l)
*   [Is "triggerer" correct, or is there some other word to identify the person who triggered something?](https://ell.stackexchange.com/questions/274203/is-triggerer-correct-or-is-there-some-other-word-to-identify-the-person-who-t)
*   [Why was the Balrog beneath Moria](https://scifi.stackexchange.com/questions/242773/why-was-the-balrog-beneath-moria)
*   [Why is "Onus" in the Dative Case?](https://latin.stackexchange.com/questions/15340/why-is-onus-in-the-dative-case)
*   [A riddle for the highway](https://puzzling.stackexchange.com/questions/107111/a-riddle-for-the-highway)
*   [Is it safe to use #ifdef guards on C++ class member functions?](https://stackoverflow.com/questions/66040598/is-it-safe-to-use-ifdef-guards-on-c-class-member-functions)
*   [What is a good approach to handling exceptions?](https://softwareengineering.stackexchange.com/questions/421756/what-is-a-good-approach-to-handling-exceptions)
*   [Are the sticks of RAM in my desktop computer volatile? Is it safe to sell them?](https://security.stackexchange.com/questions/244255/are-the-sticks-of-ram-in-my-desktop-computer-volatile-is-it-safe-to-sell-them)
*   [In the UK, can a landlord/agent add new tenants to a joint tenancy agreement without the consent of the current tenants?](https://law.stackexchange.com/questions/60814/in-the-uk-can-a-landlord-agent-add-new-tenants-to-a-joint-tenancy-agreement-wit)
*   [Why does this script running su never seem to terminate if I change user inside the script?](https://askubuntu.com/questions/1313550/why-does-this-script-running-su-never-seem-to-terminate-if-i-change-user-inside)
*   [240 dryer outlet is reading 245](https://diy.stackexchange.com/questions/215663/240-dryer-outlet-is-reading-245)
*   [Why do banks have capital requirements on deposits?](https://quant.stackexchange.com/questions/60943/why-do-banks-have-capital-requirements-on-deposits)
*   [Do missiles need anti-icing?](https://aviation.stackexchange.com/questions/84030/do-missiles-need-anti-icing)
*   [Why do Space X starship launches need permission from the FAA?](https://space.stackexchange.com/questions/49891/why-do-space-x-starship-launches-need-permission-from-the-faa)
*   [Is there a way to see linear and surface charge density as a "special case" of volume charge density?](https://physics.stackexchange.com/questions/612432/is-there-a-way-to-see-linear-and-surface-charge-density-as-a-special-case-of-v)
*   [How does everyone not become poor over time?](https://economics.stackexchange.com/questions/42416/how-does-everyone-not-become-poor-over-time)
*   [Is calling a character a "lunatic" or "crazy" ableist when it is in reference to their erratic behavior?](https://writing.stackexchange.com/questions/54802/is-calling-a-character-a-lunatic-or-crazy-ableist-when-it-is-in-reference-to)
*   [Did Alastor Moody know what name others used for him?](https://scifi.stackexchange.com/questions/242765/did-alastor-moody-know-what-name-others-used-for-him)
*   [Comparing 'upright-ness' across several endurance road bikes](https://bicycles.stackexchange.com/questions/74882/comparing-upright-ness-across-several-endurance-road-bikes)

[more hot questions](#)

[Question feed](/feeds/question/49918313 "Feed of this question and its answers")

# Subscribe to RSS

Question feed

To subscribe to this RSS feed, copy and paste this URL into your RSS reader.



[](#)

<div> <img src="/posts/49918313/ivc/0af4" class="dno" alt="" width="0" height="0"> </div>

lang-yaml
