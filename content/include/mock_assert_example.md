    // ...
    t.Run("nominal case", func(t *testing.T) {
        t.Parallel()

        mock := mocks.BaselineTransport(t)
        mock.Request = func(method, url string) (*http.Response, error) {
            assert.Equal(t, http.MethodPost, method)
            assert.Equal(t, "https://google.com", url)

            return testResponse, nil
        }

        subject := NewServer(mock)

        got, err := subject.Call()
        require.NoError(t, err)
        assert.NotNil(t, got)
        // Assertions...
    })
    t.Run("handles failure to make request", func(t *testing.T) {
        t.Parallel()

        mock := mocks.BaselineTransport(t)
        mock.Request = func(string, string) (*http.Response, error) {
            return nil, mocks.GenericError()
        }

        subject := NewServer(mock)

        got, err := subject.Call()
        assert.Error(t, err)
    })
    // ...