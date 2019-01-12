# confyg

A suitably generic form of the Go module configuration file parser

Usage is simple:

```go
type server struct {
	port string
	keys *crypto.Keypair
	db   *storm.DB
}

func (s *server) Allow(verb string, block bool) bool {
	switch verb {
	case "port":
		return !block
	case "dbfile":
		return !block
	case "keys":
		return !block
	}

	return false
}

func (s *server) Read(errs *bytes.Buffer, fs *confyg.FileSyntax, line *confyg.Line, verb string, args []string) {
	switch verb {
	case "port":
		_, err := strconv.Atoi(args[0])
		if err != nil {
			fmt.Fprintf(errs, "%s:%d value is not a number: %s: %v\n", fs.Name, line.Start.Line, args[0], err)
			return
		}

		s.port = args[0]

	case "dbfile":
		dbFile := args[0][1 : len(args[0])-1] // shuck off quotes

		db, err := storm.Open(dbFile)
		if err != nil {
			fmt.Fprintf(errs, "%s:%d failed to open storm database: %s: %v\n", fs.Name, line.Start.Line, args[0], err)
			return
		}

		s.db = db

	case "keys":
		kp := &crypto.Keypair{}

		pubk, err := hex.DecodeString(args[0])
		if err != nil {
			fmt.Fprintf(errs, "%s:%d invalid public key: %v\n", fs.Name, line.Start.Line, err)
			return
		}

		privk, err := hex.DecodeString(args[1])
		if err != nil {
			fmt.Fprintf(errs, "%s:%d invalid private key: %v\n", fs.Name, line.Start.Line, err)
			return
		}

		copy(kp.Public[:], pubk[0:32])
		copy(kp.Private[:], privk[0:32])

		s.keys = kp
	}
}
```
