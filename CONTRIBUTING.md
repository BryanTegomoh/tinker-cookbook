# Development

## Organization of training scripts

We're designing the codebase with the following goals:

1. Low barrier to entry: it should be dead simple to run something and see numbers go up.
2. Extensible: it should be possible to pass in custom datasets and evals and control all the hyperparameters.
3. Science-friendly: it should be easy to run sweeps, and analyze the results.

To achieve this, we'll use the following structure around training scripts:

- There's a main training function, such as `sft.py` or `rl_bandit/train.py`, which contains the main loop.
    - This function contains a detailed config object (`Config`), which isn't constructable from the command line.
    - The config contains members that specify things like datasets and evals. However, these should be chz configs (with a `.build` method that constructs the actual object) or callables (we recommend using functools.partial). This way, the config is serializable, which is useful for sweeps.
- There's an auxiliary script, called something like `sft_cli.py` or `rl_bandit/train_cli.py`, which contains a smaller config object (`CLIConfig`), which is constructable from the command line. This script is useful to let people get started with the library, without digging into a lot of code and learning about new classes.

## Async

Async is very useful for RL, where it allows us to make many queries in parallel (e.g., sampling calls). For all of the interfaces used in RL (such as the `Env` class), all the methods that take nontrivial amounts of time should be async. For some of the other code, such as `sft.py`, we've chosen not to use async methods, just to make it more beginner-friendly, as many python programmers are not familiar with async.

## Typing

Please use typing wherever possible; avoid `Any` and `type: ignore`; prefer casting. However, avoid using convoluted generics or writing code that's much more verbose just to satisfy the type checker. Prefer using single types over union types.

## Classes

There are a lot of different classes, which might make the code feel less approachable. However, they follow *the builder pattern*, and the code should be less confusing when you know the pattern.

We can illustrate the pattern with the two main examples:

- A `SupervisedDatasetBuilder` is a configuration object which builds a `SupervisedDataset`.
- An `RLDatasetBuilder` is a configuration object which builds an `RLDataset`, which generates batches of `EnvGroupBuilder` objects, which each generate a group of `Env` objects.

Here, the `SupervisedDatasetBuilder`, `RLDatasetBuilder`, and `EnvGroupBuilder` are all configuration objects, which have a `__call__` method that builds another object. You can see these objects in [supervised/types.py](tinker_cookbook/supervised/types.py) and [rl/types.py](tinker_cookbook/rl/types.py).

In general, we use a lot of configuration objects, with a `__call__` method that returns a heavyweight object (like a dataset). We use `chz` for the configuration objects -- it's similar to a dataclass but with some extra features that are nice for configs. We use either dataclasses or regular python classes for the heavyweight objects.

## Envs

An `Env` is an RL environment. For those with an RL background, it roughly corresponds to an MDP or a POMDP, however we use in more general cases (such as multi-agent settings) that don't strictly correspond to the MDP/POMDP formalism. It's roughly analogous the concept of an Env in OpenAI Gym, but unlike OpenAI Gym, we don't have a `reset` method; rather, the env should be discarded after a rollout. Any shared resources should be maintained by whatever object is creating the envs.

The `Env`s are created by `EnvGroupBuilder`s. The group of envs returned by `EnvGroupBuilder` have something in common; either they correspond to the same task (in which case we can use this information for variance reduction, as in GRPO, which centers per group); or, we can use the group to define a multi-agent environment.

- One common multi-agent environment is where we use a pairwise preference model to compare pairs of completions.
- We can also use the group to define a two-player game. Some two player games such as tic-tac-toe are currently supported through the [textarena](tinker_cookbook/rl/textarena_envs.py) environments.


## Notation

We'll use subscripts to indicate the shapes of objects. For example, `tokens_P_G_T` indicates a three-dimensional array of tokens, with `P` problems, `G` groups, and `T` tokens per groups, so `tokens_P_G_T[p][g][t]` should refer to a single token. In many cases, the arrays will be ragged. E.g., the `T` axis will have different lengths for different `(p,g)`. Sometimes, a given dimension will be flattened from two dimensions. If we write `tokens_PG_T`, that means that we have a two dimensional array, where the 0th dimension is flattened from the `P` and `G` dimensions.

### Common Dimension Names

Here are the standard dimension subscripts used throughout the codebase:

- `_D`: Data/Datum dimension (for training data items)
- `_G`: Group dimension (for multiple attempts/rollouts of the same problem)
- `_P`: Problem dimension (for different problems/prompts)
- `_T`: Token/Time dimension (for sequences)

The relationship between dimensions in RL:
- A batch contains multiple problems (`_P`)
- Each problem spawns multiple attempts/environments (`_G`), forming a group
- Each attempt produces one trajectory
- Advantages are normalized within each group (across the `_G` dimension)

Examples:
- `env_group_builders_P`: A list of environment builders, one per problem
- `trajectories_G`: Multiple trajectories from attempts at the same problem
- `rewards_G`: Rewards for each attempt within a group
- `tokens_P_G_T`: Tokens with problem, group, and time dimensions
- `data_D`: A list of training data items

## Testing

TODO(tianyi): add testing info

# Call for Proposals

TODO(tianyi): add
