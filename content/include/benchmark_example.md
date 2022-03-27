// test_trie.go
func BenchmarkTrie_InsertMany(b *testing.B) {

	paths, payloads := helpers.SampleRandomRegisterWrites(helpers.NewGenerator(), 1000)

	b.Run("insert 1000 elements (reference)", func(b *testing.B) {
		for i := 0; i < b.N; i++ {
			ref := reference.NewEmptyMTrie()
			ref, _ = reference.NewTrieWithUpdatedRegisters(ref, paths, payloads)
			_ = ref.RootHash()
		}
	})

	b.Run("insert 1000 elements (new)", func(b *testing.B) {
		for i := 0; i < b.N; i++ {
			tr := trie.NewEmptyTrie()
			tr, _ = tr.Mutate(paths, payloads)
			_ = tr.RootHash()
		}
	})
}