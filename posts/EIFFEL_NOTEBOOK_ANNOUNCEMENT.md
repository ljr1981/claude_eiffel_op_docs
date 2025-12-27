# Eiffel Notebook 1.0.0-alpha.34 - Multi-Class Support with Multiple Inheritance

I'm pleased to announce a significant update to Eiffel Notebook: **multi-class support with full multiple inheritance**.

## What is Eiffel Notebook?

Eiffel Notebook is a Jupyter-like REPL for Eiffel. Write Eiffel code in cells, execute them, and see results immediately - all using natural Eiffel syntax.

Originally suggested by Javier Velilla and designed with guidance from Eric Bezault (cell classification using natural Eiffel syntax), this project is part of the [Simple Eiffel](https://github.com/simple-eiffel) ecosystem.

## What's New in Alpha 34

### Multi-Class with Multiple Inheritance

You can now define complete Eiffel classes and combine them using multiple inheritance - something unique for a REPL:

```
e[1]> -class CAR
class CAR
feature
    drive do print ("Driving on road%N") end
end

e[2]> -class BOAT
class BOAT
feature
    sail do print ("Sailing on water%N") end
end

e[3]> -class CAR_BOAT
class CAR_BOAT
inherit
    CAR
    BOAT
feature
    amphibious_mode do
        print ("Entering water...%N")
        sail
        print ("Back on land...%N")
        drive
    end
end

e[4]> v: CAR_BOAT
e[5]> create v
e[6]> v.amphibious_mode
Entering water...
Sailing on water
Back on land...
Driving on road
```

### Edit Existing Classes

Made a mistake? Use `-class NAME` again to edit:

```
e[7]> -class CAR
Editing class CAR (cell 1):
class CAR
feature
    drive do print ("Driving on road%N") end
end

Type complete new class (starts with 'class CAR'):
class CAR
feature
    drive do print ("Vroom! Driving on road%N") end
    honk do print ("Beep beep!%N") end
end

Class CAR updated in cell 1.
```

### Silent Compile Default

The notebook now starts in silent compile mode for a cleaner REPL experience. Use `-compile verbose` if you need to see compiler output.

## Key Features

- **Natural Eiffel syntax** - No special keywords, write standard Eiffel
- **Multi-class support** - Define complete classes with `-class NAME`
- **Multiple inheritance** - Full MI support
- **Design by Contract** - Full require/ensure/invariant support
- **Melt mode** - 10-30x faster execution after initial compile
- **Session persistence** - Save/restore notebook sessions
- **Cross-platform** - Windows, Linux, WSL2

## Installation

### Windows
Download the installer: [eiffel_notebook_setup_1.0.0-alpha.34.exe](https://github.com/simple-eiffel/simple_notebook/releases/tag/v1.0.0-alpha.34)

### Linux/WSL2
Build from source:
```bash
git clone https://github.com/simple-eiffel/simple_notebook
cd simple_notebook
ec -batch -config simple_notebook.ecf -target notebook_cli -c_compile
./EIFGENs/notebook_cli/W_code/simple_notebook
```

## Links

- **GitHub**: https://github.com/simple-eiffel/simple_notebook
- **Documentation**: https://simple-eiffel.github.io/simple_notebook/
- **User Guide**: https://simple-eiffel.github.io/simple_notebook/user-guide.html

## Acknowledgments

- **Eric Bezault** (Gobo Eiffel) - Cell classification design
- **Javier Velilla** - Original project idea

Feedback and contributions welcome!

Larry