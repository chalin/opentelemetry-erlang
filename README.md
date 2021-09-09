OpenTelemetry
=====

[![EEF Observability WG project](https://img.shields.io/badge/EEF-Observability-black)](https://github.com/erlef/eef-observability-wg)
[![Hex.pm](https://img.shields.io/hexpm/v/opentelemetry)](https://hex.pm/packages/opentelemetry)
![Build Status](https://github.com/open-telemetry/opentelemetry-erlang/workflows/Common%20Test/badge.svg)

OpenTelemetry stats collection and distributed tracing framework for Erlang.

Requires OTP 21.3 or above.

## Contacting Us

We hold regular meetings. See details at [community page](https://github.com/open-telemetry/community#special-interest-groups).

We use [GitHub Discussions](https://github.com/open-telemetry/opentelemetry-erlang/discussions) for support or general questions. Feel free to drop us a line.

We are also present in the #otel-erlang-elixir channel in the [CNCF slack](https://slack.cncf.io/). Please join us for more informal discussions.

## Design

The [OpenTelemetry specification](https://github.com/open-telemetry/opentelemetry-specification) defines a language library as having 2 components, the API and the SDK. The API must not only define the interfaces of any implementation in that language but also be able to function as a noop implementation of the tracer. The SDK is the default implementation of the API that must be optional.

When instrumenting a project your application should only depend on the
[OpenTelemetry API](https://hex.pm/packages/opentelemetry_api) application,
found in directory `apps/opentelemetry_api` of this repo which is published as
the hex package [opentelemetry_api](https://hex.pm/packages/opentelemetry_api). 

This repository is the Erlang's SDK implementation and should be included in the final release and configured to setup the sampler, span processors and span exporters.

## Usage

### Hex Dependencies

It is recommended to use the versions published on hex.pm for [OpenTelemetry
API](https://hex.pm/packages/opentelemetry_api) and [OpenTelemetry
SDK](https://hex.pm/packages/opentelemetry).

### Git Dependencies

Because the OpenTelemetry OTP Applications are kept in a single repository,
under the directory `apps`, either [rebar3's](https://rebar3.org) `git_subdir`
(rebar 3.14 or above is required) or
[mix's](](https://elixir-lang.org/getting-started/mix-otp/introduction-to-mix.html))
`sparse` feature must be used when using as Git dependencies in a project. The
blocks below shows how in rebar3 and mix the git repo for the API and/or SDK
Applications can be used.

``` erlang
{opentelemetry_api, {git_subdir,
"http://github.com/open-telemetry/opentelemetry-erlang", {branch, "main"}, "apps/opentelemetry_api"}}
{opentelemetry, {git_subdir,
"http://github.com/open-telemetry/opentelemetry-erlang", {branch, "main"}, "apps/opentelemetry"}}
```

``` elixir
{:opentelemetry_api, github: "open-telemetry/opentelemetry-erlang", sparse:
"apps/opentelemetry_api", override: true},
{:opentelemetry, github: "open-telemetry/opentelemetry-erlang", sparse: "apps/opentelemetry", override: true},
```

The `override: true` is required because the SDK Application, `opentelemetry`, has
the API in its `deps` list of its `rebar.config` as a hex dependency and this will
clash when `mix` tries to resolve the dependencies and fail without the
override. `override: true` is also used on the SDK because the
`opentelemetry_exporter` application depends on it and the API as hex deps so if
it is included the override is necessary.

### Including in Release

In an Erlang project add `opentelemetry` as the first element of the release's applications:

``` erlang
{relx, [{release, {<name>, <version>}, 
         [{opentelemetry, temporary},
          <your applications>]},
        ...]}.
```

In the above example `opentelemetry` is set to `temporary` so that if the `opentelemetry` application crashes, or is shutdown, it does not terminate the other applications in the project. This is optional, the `opentelemetry` application purposely sticks to `permanent` for the processes started by the root supervisor to leave it up to the end user whether they want the crash or shutdown or `opentelemetry` to be ignored or cause the shutdown of the rest of the applications in the release.

Doing the same for an Elixir project would be:

``` elixir
def project do
  [
    ...
    releases: [
      <name>: [
        applications: [opentelemetry: :temporary]
      ],

      ...
    ]
  ]
end
```

## Benchmarks

Running benchmarks is done with [benchee](https://github.com/bencheeorg/benchee). Benchmark functions are in modules under `samples/`. To run them open a rebar3 shell in the `bench` profile:

``` shell
$ rebar3 as bench compile

> otel_benchmarks:run().
```

If an Elixir script is wanted for the benchmarks they could be run like:

``` shell
$ ERL_LIBS=_build/bench/lib/ mix run --no-mix-exs samples/run.exs
```

## W3C Trace Context Interop Tests

Start the interop web server in a shell:

``` shell
$ rebar3 as interop shell

> w3c_trace_context_interop:run().
```

Then, clone the [W3C Trace Context repo](https://github.com/w3c/trace-context) and run the tests:

``` shell
$ cd test
$ python3 test.py http://127.0.0.1:5000/test
```

## Contributing

Approvers:
- [Fred Hebert](https://github.com/ferd), Honeycomb

Maintainers:
- [Łukasz Jan Niemier](https://github.com/hauleth), Remote Technology, Inc.
- [Iliia Khaprov](https://github.com/deadtrickster), VMWare
- [Tristan Sloughter](https://github.com/tsloughter)
