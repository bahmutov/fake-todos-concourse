# Example Concourse CI definition

Concourse CI has very low level API, thus even simple things
like "npm install && npm test" become complicated. This is an example
repo with a shell script executing `npm install && npm test` command
and a Concourse pipeline definition file.

The definition imports two repos: this repo holds the CI files,
and the actual source repo. The two input resources as they are called
are placed into `scripts` and `repo` folders. Then we execute the
shell script and everything works

```yaml
resources:
- name: fake-todos-git
  type: git
  source:
    uri: https://github.com/bahmutov/fake-todos.git
    branch: master
- name: fake-todos-concourse-git
  type: git
  source:
    uri: https://github.com/bahmutov/fake-todos-concourse.git
    branch: master
```

Job definition

```yaml
jobs:
- name: fake-todos-test
  public: true
  plan:
    - get: fake-todos-git
    - get: fake-todos-concourse-git
    - task: test-fake-todos
      config:
        platform: linux
        image: docker:///mhart/alpine-node
        inputs:
          - name: fake-todos-git
            path: repo
          - name: fake-todos-concourse-git
            path: scripts
        run:
          path: ../scripts/npm-install-and-test.sh
          dir: repo
```

To import the pipeline definition into your Concourse CI server (after login)
under alias `target`

```sh
fly sp -t <target> -c pipeline.yml -p pipeline
```

## Extra Concourse CI resources

- https://github.com/starkandwayne/concourse-tutorial
- http://danoncoding.com/programming/2015/12/17/setting-up-a-simple-pipeline-with-concourse-ci/
