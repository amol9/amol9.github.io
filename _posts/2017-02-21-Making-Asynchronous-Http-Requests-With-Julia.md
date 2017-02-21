---
layout: post
title: Making Asynchronous Http Requests With Julia
tags: julia, http, asynchronous
---

Like I wrote in my [previous post](/2017/02/21/The-Julia-Programming-Language/), julia has some powerful parallel computing capabilities built right into the language. So, I decided to give one of them a run.

I used @async and @sync to asynchronously do http requests and wait for all of them to complete respectively.

The program has a dependency on package: [Requests](https://github.com/JuliaWeb/Requests.jl). You can install it using:

{% highlight julia %}
> Pkg.add("Requests")
{% endhighlight %}

The asynchronous workhorse of the program is this section:

{% highlight julia %}
@sync begin
	for package in packages
		@async begin
			j = json(get(make_pypi_url(package)))
			d, w, m = parse_json(j)
			o = @sprintf("%-15s: %7d %7d %7d", package, d, w, m)
			println(o)
		end
	end
end
{% endhighlight %}

"@async begin - end" wraps the part to be executed asynchronously. Note that I did not bother to read each individual get() requests asynchronously. I didn't have to. Since, as soon as any of these http requests blocks, the corresponding task blocks and the task manager starts executing the next task, i.e., another http request.

Initially, I used the macro @printf to format and print the output. But, that call is not lock synchronized. So, it mixes up the output from multiple tasks. After searching a little, I found that the function, println(), uses locks when writing to asynchronous streams. I changed my code to the above and it worked.

The last bit to note in the above block is that the for loop is wrapped in a "@sync begin - end" block. That makes the program wait until all the enclosing asynchronous tasks are complete, i.e., In our case, all the http requests are done, parsed and output printed. Without this, the program would just run to its termination without waiting for asynchronous tasks to complete.

I used the json() function provided by the "Requests" library. There is a built in package for json parsing in julia, JSON. You can use that as well.

The "Requests" library has another version of get(), get_streaming(). It lets you read the response asynchronously. If you want more control over reading http response, this function is your friend.

One pleasant thing about julia is that the string formatting notation is very similar to that of python. For the output of this program, the format string was exactly like python. It helps when you don't have to struggle because someone decided to reinvent the wheel for trivial things.

The program is a mere 42 lines long, and a lot of them are just the "end" statements. This shows the asynchronous programming prowess of julia.

For editing, I used VS Code with [julia language extension](https://github.com/JuliaEditorSupport/julia-vscode).

{% highlight julia %}
import Requests: get, json

base_url = "https://pypi.python.org/pypi/"
package_list_file = "packages.txt"

make_pypi_url = p::String -> string(base_url, p, "/json")

function parse_json(j::Dict)::Tuple{Int, Int, Int}
    dls = j["info"]["downloads"]
    dls["last_day"], dls["last_week"], dls["last_month"]
end

function get_stats()
    packages = strip.(read_package_list())
    @printf("%-15s: %7s %7s %7s\n", "package", "day", "week", "month")

    @sync begin         # will wait for all enclosed async tasks to complete
        for package in packages
            @async begin
                j = json(get(make_pypi_url(package)))
                d, w, m = parse_json(j)
                o = @sprintf("%-15s: %7d %7d %7d", package, d, w, m)
                println(o)          # have to sprintf to a string and then println to avoid intermingled output
            end
        end
    end
end

function read_package_list()::Array{String}
    try
        readlines(open(package_list_file))
    catch e
        println("error opening packages.txt")
        quit()
    end
end

function main()
    get_stats()
end

main()
{% endhighlight %}

[code in github repo](https://github.com/amol9/scripts/blob/master/pypi/julia/pypi.jl)
