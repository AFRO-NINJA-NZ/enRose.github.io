# Rule Engine Pattern

**29 Jan 2020**

Before delving into why and what the pattern is, it is important to know where the inspiration is drawn from - [Unix Rule of Separation](https://enrose.github.io/c-sharp/unix-rule-of-separation).

## Motivation


## Usage

```c#
namespace Barin.RuleEngineExample
{
    // For DI to work, we need an interface.
    public interface ICatRuleCtx {}

    public class CatRuleCtx : ICatRuleCtx, IV8RuleCtx
    {
        public Guid Id { get; set; }
        public Guid OwnerId { get; set; }
        public List<Owner> Owners { get; set; }
        public Owner GoodOwner { get; set; }
        public string FailedAt { get; set; }

        [Dependency]
        public ICatRepository CatRepository { get; set; }
    }

    public class CatRules : V8Rule<CatRuleCtx>
    {
        public CatRules(CatRuleCtx ctx)
        {
            Ctx = ctx;

            Rules = new Func<Task<bool>>[] {
                LinkedOwnersRule,
                MustLoveCatRule
            };
        }

        public async Task<bool> LinkedOwnersRule()
        {
            var owners = await Ctx.CatRepository.GetOwners(
                Ctx.Id
            )
            .ConfigureAwait(false);

             Ctx.Owners = owners;

            return owners?.Count > 0;
        }

        public Task<bool> MustLoveCatRule()
        {
            var goodOwner = Ctx.Owners.FirstOrDefault(
                x => x.attitude == "has a cats loving heart"
            );

            if (goodOwner == null)
            {
                return Task.FromResult(false);
            }

            Ctx.GoodOwner = goodOwner;

            return Task.FromResult(true);
        }
    }
}
```

```c#
var ctx = new CatRuleCtx();
var catRules = new CatRules(ctx);
var isValid = await catRules.Run().ConfigureAwait(false);
```
