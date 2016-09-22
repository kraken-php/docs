# Adding Commands

- [Introduction](#introduction)
- [Adding Commands](#adding-commands)

<a name="introduction"></a>
## Introduction

<a name="adding-commands"></a>
### Adding Commands

Adding commands that can be invoked using Kraken Client and executed on any given runtime container is easy, but multi-step process. This short article will show you how it can be done implementing simple command that returns container's name.

#### Creating Container's Command

The first step is to create command responder on container's side. Let's assume this command will be called `App\Runtime\Command\Test\TestCommand`.

    use Kraken\Runtime\Command\Command;
    use Kraken\Runtime\Command\CommandInterface;
    
    class TestCommand extends Command implements CommandInterface
    {
        protected function command($params = [])
        {
            return $this->runtime->getAlias();
        }
    }

#### Registering Container's Command

After creating command, you have to register it. This can be done in several ways.

One of the way of registering command is to use container's `config.php` file and its `command.models` and `command.commands` values:

    // this declaration will me your command craftable using Command.Factory service.
    'models' => [
        'App\Runtime\Command\Test\TestCommand'
    ]

    // this declaration will automatically attach your command to runtime.
    'commands' => [
        'app:test' => 'App\Runtime\Command\Test\TestCommand'
    ]

The second method is to inject them directly to command services:

    $manager = $container->make('Command.Manager');
    $runtime = $container->make('Runtime');
    
    $manager->set('app:test', new TestCommand([ 'runtime' => $runtime ]));

#### Creating Console's Command

If you are already in this step, it means your command is already registered, but it is not visible from Kraken Console level. To add invoker to console, create class `App\Console\Command\Test\TestCommand`:

    use Kraken\Console\Client\Command\Command;
    use Symfony\Component\Console\Input\InputInterface;
    use Symfony\Component\Console\Output\OutputInterface;
    
    class TestCommand extends Command
    {
        protected function config()
        {
            $this
                ->setName('app:test')
                ->setDescription('This is my first command.')
            ;
            $this->addArgument(
                'alias',
                InputArgument::REQUIRED,
                'Alias of thread to ask.'
            );
        }
        
        protected function command(InputInterface $input, OutputInterface $output)
        {
            return $this->informServer($input->getArgument('alias'), 'app:test');
        }
    }

#### Registering Console's Command

To register previously created command into console, open console `config.php` file and add it to `command.models` list.

    // this declaration will me your command invokable using Kraken Console.
    'models' => [
        'App\Console\Command\Test\TestCommand'
    ]
