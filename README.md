# westailabs.com

Source for the public [West AI Labs](https://westailabs.com) website: the landing
page and long-form research articles. Built with Jekyll and deployed to GitHub
Pages by `.github/workflows/deploy.yml`.

```bash
bundle install
bundle exec jekyll serve   # http://localhost:4000
bundle exec jekyll build   # output in _site/
```

Contributor and agent instructions live in [AGENTS.md](AGENTS.md) and
[CONTRIBUTING.md](CONTRIBUTING.md).

## License

This repository is dual-licensed by type of material:

- **Content** (research articles in `_research/`, page copy, and images) is
  licensed under [Creative Commons Attribution 4.0 (CC BY 4.0)](https://creativecommons.org/licenses/by/4.0/).
  See [LICENSE-CONTENT](LICENSE-CONTENT). Reuse is welcome with attribution to
  West AI Labs LLC and a link to the original article.
- **Code** (templates, CSS, JavaScript, Jekyll config, workflows) is licensed
  under the [MIT License](LICENSE).

The West AI Labs name and logos are trademarks and are not covered by either
license.
