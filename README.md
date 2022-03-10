# flux-multi-stage
Flux experiments
 Link =[Link](https://www.awstutorials.cloud/post/tutorials/flux-multi-stage/)

fluxctl install \
--git-user=${GHUSER} \
--git-email=${GHUSER}@users.noreply.github.com \
--git-path=acpt \
--git-branch=main \
--manifest-generation=true \
--git-url=git@github.com:${GHUSER}/flux-multi-stage \
--namespace=flux | kubectl apply -f -



fluxctl install \
--git-user=${GHUSER} \
--git-email=${GHUSER}@users.noreply.github.com \
--git-path=acpt \
--git-branch=main \
--manifest-generation=true \
--git-url=git@github.com:${GHUSER}/flux-multi-stage \
--namespace=flux | kubectl delete -f -

fluxctl identity --k8s-fwd-ns flux