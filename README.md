<h1 align="center">Blockscout</h1>
<p align="center">Blockchain Explorer for inspecting and analyzing EVM Chains.</p>
<div align="center">
fix specific past block sync errors


v1.0    apps/indexer/lib/indexer/transform/token_transfers.ex
230
   231    # ERC-20 token transfer
   
   # [▼▼▼ 여기에 아래 한 줄을 추가하십시오 ▼▼▼]
   defp parse_params(_), do: nil

   232    defp parse_params(%{second_topic: second_topic, third_topic: third_topic, fourth_topic: nil} = log)
   233          when not is_nil(second_topic) and not is_nil(third_topic) do
추가해서 토큰 및 멀티시그 트랜스퍼 관련 에러들 무시

v.1.1  apps/indexer/lib/indexer/transform/token_transfers.ex
토큰 무시는 안하고 에러나는것만 무시로 변경
# [삭제] 이 줄을 지우세요.
defp parse_params(_), do: nil

# apps/indexer/lib/indexer/transform/token_transfers.ex

  defp do_parse(log, %{tokens: tokens, token_transfers: token_transfers} = acc, type \\ :erc20_erc721) do
    try do
      parse_result =
        case type do
          :erc1155 -> parse_erc1155_params(log)
          :erc404 -> parse_erc404_params(log)
          _ -> parse_params(log)
        end

      case parse_result do
        {token, token_transfer} ->
          %{
            tokens: [token | tokens],
            token_transfers: [token_transfer | token_transfers]
          }

        nil ->
          acc
      end
    rescue
      # 모든 에러를 잡아서 무시하고 넘어감 (로그만 남김)
      e ->
        Logger.warn("Skipping malformed token transfer: #{inspect(e)}")
        acc
    catch
      # catch까지 사용하여 확실하게 방어
      _, _ -> acc
    end
  end
