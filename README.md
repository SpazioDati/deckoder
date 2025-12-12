# deckoder

Check files in docker image

fork from [aquasecurity/fanal](https://github.com/aquasecurity/fanal)

## Feature

- Fetch target image data if there is no image in local
- Check target condition files

## Example

See [`cmd/deckoder/`](cmd/deckoder/main.go)

```go
func main() {
	if err := run(); err != nil {
		log.Fatalf("%+v", err)
	}
}

func run() (err error) {
	ctx := context.Background()
	tarPath := flag.String("f", "-", "layer.tar path")
	flag.Parse()

	args := flag.Args()

	opt := types.DockerOption{
		Timeout:  600 * time.Second,
		SkipPing: true,
	}

	var ext extractor.Extractor
	var cleanup func()
	if len(args) > 0 {
		ext, cleanup, err = docker.NewDockerExtractor(ctx, args[0], opt)
		if err != nil {
			return err
		}
	} else {
		ext, cleanup, err = docker.NewDockerArchiveExtractor(ctx, *tarPath, opt)
		if err != nil {
			return err
		}
	}
	defer cleanup()

	filter := utils.CreateFilterPathFunc([]string{"etc/shadow"})
	ac := analyzer.New(ext)
	fileMap, err := ac.Analyze(ctx, filter)
	if err != nil {
		return err
	}

	for name, f := range fileMap {
		fmt.Println(name, string(f.Body))
	}
	return nil
}
```

## Notes

When using `latest` tag, that image will be cached. After `latest` tag is updated, you need to clear cache.
