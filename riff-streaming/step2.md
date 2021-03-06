Let's now create a averager function that will read a stream of cloud events and emits an average of the input numbers across multiple invocations until a negative number is encountered. Copy the code below to the editor on the right.
<pre class="file" data-filename="Averager.java" data-target="replace">package com.acme;

import java.util.function.Function;
import reactor.core.publisher.Flux;
import reactor.math.MathFlux;

public class Averager implements Function&lt;Flux&lt;Float&gt;, Flux&lt;Float&gt;&gt; {

	@Override
	public Flux&lt;Float&gt; apply(Flux&lt;Float&gt; floatFlux) {
		return floatFlux.windowWhile(aFloat -> aFloat > 0.0f).flatMap(MathFlux::averageFloat);
	}
}
</pre>

riff's [streaming-processor](https://github.com/projectriff/streaming-processor) will read the input stream of cloud events and [java-function-invoker](https://github.com/projectriff/java-function-invoker) will invoke the function by converting the input into a [Flux](https://projectreactor.io/docs/core/release/api/reactor/core/publisher/Flux.html). Similarly, the output of the function is converted into cloud events and published to the output stream.

We will now build a container with a language runtime and a riff invoker for this function. We need a registry to push the container image that is built by riff. You can use dockerhub, gcr or any other container registry. This tutorial is running a local registry. We will set a environment variable to point to the registry

`export REGISTRY=[[HOST_SUBDOMAIN]]-5000-[[KATACODA_HOST]].environments.katacoda.com`{{execute}}

Now, provide the registry credentials so that riff can push the built images:
`riff credentials apply my-reg --registry http://$REGISTRY --default-image-prefix $REGISTRY`{{execute}}
In our case there are no credentials, only configure the registry URL.

Build the function using `riff function create averager --local-path ./averager --handler com.acme.Averager --tail`{{execute}}

The logs here show that the programming language was detected as Java and the appropriate runtime and invoker layer were added to build a container.
