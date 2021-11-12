    package mocks

    import (
        "testing"

        "github.com/onflow/flow-go/ledger/complete/mtrie/trie"
    )

    type Loader struct {
        TrieFunc func() (*trie.MTrie, error)
    }

    func BaselineLoader(t *testing.T) *Loader {
        t.Helper()

        l := Loader{
            TrieFunc: func() (*trie.MTrie, error) {
                return GenericTrie, nil
            },
        }

        return &l
    }

    func (l *Loader) Trie() (*trie.MTrie, error) {
        return l.TrieFunc()
    }